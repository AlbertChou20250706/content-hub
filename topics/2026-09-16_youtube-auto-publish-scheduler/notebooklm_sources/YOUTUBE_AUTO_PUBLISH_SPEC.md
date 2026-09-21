# YouTube 影片排程自動發布系統 — 技術規格書（NotebookLM 原始素材）

> 這份文件是使用者原始撰寫的技術規格書全文（v1.2），直接存檔作為 NotebookLM 餵入用的原始技術文件。
> 之後若錄製 YouTube 逐字稿產出，可視情況保留本檔作為原始素材存檔，或由逐字稿取代。
> README.md 為濃縮後的技術規劃紀錄，完整細節（含虛擬碼、資料結構、API 設定步驟）請見本檔。

---

| | |
|---|---|
| 專案代號 | ChouAP.Cloud - YT-AutoPublish |
| 文件版本 | v1.2 |
| 撰寫日期 | 2026-09-16 |
| 執行環境 | GitHub Actions（雲端排程，本機電腦免開機）|
| 目標使用者 | Albert Chou（個人自動化）|

### 版本歷史

| 版本 | 日期 | 修改內容 | 設計者 |
|---|---|---|---|
| v1.0 | 2026-09-16 | 初版定案：每次觸發最多發 1 支 | Albert Chou |
| v1.1 | 2026-09-16 | 發布節奏改為「每次觸發批次清空佇列」，同步調整架構圖、主流程邏輯、通知時機 | Albert Chou |
| v1.2 | 2026-09-16 | 釐清舊影片（已是 public）可與新影片（unlisted）混放同一佇列，統一走同套處理流程；`videos.update` 明定為冪等操作 | Albert Chou |

---

## 0. 必要前置修正（實作前務必確認，否則機制空轉）

### 0-1. 待發布影片的隱私狀態

> **新上傳、要享有「排程延遲公開」效果的影片，上傳時建議設為「未列出（Unlisted）」，不可以是「公開（Public）」。**

自動化的核心動作是「呼叫 API 把 `privacyStatus` 設為 `public`」。如果影片放進佇列時就已經是 public，觀眾早就看得到了，「排程延遲公開」這個效果對它沒有意義。

**但這不影響舊影片被納入同一套流程處理**：`videos.update` 設定 `privacyStatus=public` 是**冪等操作**——不論影片原本是 `unlisted` 還是本來就是 `public`，呼叫都不會出錯，只是對已公開的影片而言「狀態沒有實質變化」。因此：

- 新舊影片可以**混在同一個「待發布」播放清單**裡，依 position 順序一起被批次處理
- 舊的、本來就公開的影片，一樣會被搬到「已發布」清單、寫入 Log、算進 LINE 通知的發布清單裡——只是「公開」這個動作對它是無效果的重複設定
- 腳本**不需要**額外判斷「這支原本是不是已經 public」才決定要不要處理，統一呼叫 `videos.update(privacyStatus="public")` 即可，邏輯更單純

> ⚠️ API 實作小提醒：呼叫 `videos.update` 搭配 `part=status` 時，建議先 `videos.list` 讀出該影片目前完整的 `status` 物件，只修改 `privacyStatus` 欄位後整包送回，避免其他未指定的欄位（如 `publishAt`、`selfDeclaredMadeForKids`）被意外重置為預設值。

### 0-2. YouTube OAuth 必須切換為「Production」發佈狀態

Google OAuth 同意畫面若停留在預設的「Testing」狀態，**Refresh Token 會在 7 天後自動失效**（`invalid_grant`），對無人值守的 GitHub Actions 是致命問題（每週會靜默斷線，需要人工重新授權才能恢復）。

**正確順序**：
1. 先在 Google Cloud Console 把 OAuth 同意畫面發佈狀態改為「**In Production**」
2. 確認狀態已生效後，才進行授權流程、產生 Refresh Token
3. 把這個 Refresh Token 存進 GitHub Secrets

若順序顛倒（先產生 Token 才切 Production），該 Token 仍可能帶著 Testing 期間的限制，務必重新走一次授權取得乾淨的 Token。

（Production 狀態下，由於未走完整 Google 人工驗證，往後每次手動授權仍會看到「未驗證應用程式」警示，屬正常現象，點擊「進階」→「前往...(不安全)」繼續即可，不影響腳本日常自動執行。）

---

## 1. 需求總覽

| 項目 | 定案內容 |
|---|---|
| 觸發時間（台灣時間）| 每日 06:00 / 15:00 / 20:00 / 01:00 |
| 執行引擎 | GitHub Actions（因本機電腦常態關機，無法用 Windows工作排程器）|
| 佇列來源 | YouTube 播放清單「待發布」，依清單內順序（position）發布；**新舊影片可混放於同一清單**，不論原本是 unlisted 或已是 public，皆會被納入批次處理 |
| 已發布標記 | 影片從「待發布」播放清單移至「已發布」播放清單 + 本地 JSON 紀錄檔雙重確認 |
| 發布節奏 | **批次清空制**：每次觸發時，把當下佇列中「所有」影片依序（依 position）全部發佈完，而非只發 1 支 |
| 發布動作 | `videos.update`：統一將 `privacyStatus` 設為 `public`（冪等操作，對已公開的舊影片無實質效果但仍完整跑完流程）|
| 通知管道 | LINE Messaging API（LINE Notify 已於 2025/3/31 停用，官方建議改用此方案）|
| 通知時機 | 每次觸發結束後固定通知一次：①佇列原本就是空的 → 「本次無片可發」②佇列原本有片 → 批次處理完後回報「本次共發布 N 支」（含逐支標題清單）|
| 版本控管 | GitHub Repo（私有），佇列狀態與 Log 隨每次執行 commit，留存歷史紀錄 |
| 認證存放 | YouTube OAuth Refresh Token、LINE Channel Access Token，皆存 GitHub Secrets（加密環境變數）|

---

## 2. 系統架構圖

```
┌──────────────────────────────────────────────────────────────┐
│  GitHub Repository（Private）                                  │
│  ├─ .github/workflows/publish.yml     ← cron 排程設定           │
│  ├─ scripts/publish.py                ← 主程式                 │
│  ├─ queue/pending.json                ← 待發布佇列狀態          │
│  ├─ queue/published.json              ← 已發布紀錄（防重複）    │
│  └─ logs/                                                      │
│      ├─ publish_YYYYMMDD_HHMMSS.log   ← CLI 純文字 Log         │
│      └─ publish_YYYYMMDD_HHMMSS.html  ← HTML 折疊式 Log         │
└──────────────────────────────────────────────────────────────┘
              ↓ cron 排程觸發（UTC 時間，見第 3 節對照表）
┌──────────────────────────────────────────────────────────────┐
│  GitHub Actions Runner（雲端暫時環境，執行完即銷毀）              │
│                                                                  │
│  [1] 讀取 pending.json + 呼叫 YouTube playlistItems.list        │
│       重新核對「待發布」播放清單目前實際內容                     │
│           ↓                                                    │
│  [2] 佇列是否有影片？                                            │
│      └─ 沒有 → 呼叫 LINE API：「佇列已空，本次無片可發」          │
│                寫入 Log → 結束                                  │
│           ↓ 有                                                 │
│  [3] 依 position 順序，逐支迴圈處理佇列中「所有」影片：           │
│       ┌─────────────────────────────────────────────┐         │
│       │ 3-1 videos.update：privacyStatus → public     │         │
│       │ 3-2 playlistItems.insert（加入「已發布」清單） │         │
│       │ 3-3 playlistItems.delete（從「待發布」移除）   │         │
│       │ 3-4 更新 pending.json / published.json        │         │
│       │ 3-5 寫入該支 Log（PASS 或 ERROR，失敗不中斷，  │         │
│       │     記錄後繼續處理下一支）                     │         │
│       └─────────────────────────────────────────────┘         │
│           ↓ 迴圈跑完（佇列清空或全部嘗試過一輪）                 │
│  [4] 呼叫 LINE API：彙總本次結果                                │
│       「🎉 本次共發布 N 支：[標題清單]」                         │
│       （若有失敗項目，一併附註「M 支失敗，詳見 Log」）           │
│           ↓                                                    │
│  [5] 寫入 CLI + HTML 雙格式 Log                                  │
│           ↓                                                    │
│  [6] git commit + push 回 Repo（角色 A：狀態版控）                │
└──────────────────────────────────────────────────────────────┘
```

> **設計決定（供 Claude Code 遵循）**：批次處理中若某一支影片發布失敗（如 API 額度用盡、影片已被刪除），**不中斷整批流程**，記錄該筆為 `ERROR` 後跳過，繼續處理佇列中下一支，確保單一壞資料不會卡住整個佇列。

---

## 3. 排程時間對照表（台灣時間 UTC+8 → GitHub Actions UTC cron）

| 台灣時間 | 對應 UTC 時間 | cron 表達式 |
|---|---|---|
| 06:00 | 前一日 22:00 | `0 22 * * *` |
| 15:00 | 當日 07:00 | `0 7 * * *` |
| 20:00 | 當日 12:00 | `0 12 * * *` |
| 01:00 | 前一日 17:00 | `0 17 * * *` |

> ⚠️ GitHub Actions 的 cron 排程**不保證準時**，系統忙碌時可能延遲數分鐘到數十分鐘，這是平台已知限制，設計上需接受此誤差，不追求秒級精準。
>
> ⚠️ GitHub 規定：私有 Repo 若連續 **60 天無任何 commit 活動**，排程工作流程會被自動停用。本系統每次觸發都會 commit 佇列狀態與 Log，正常使用下不會觸發此限制；但若佇列長期空著超過 60 天沒有任何動作，需注意手動喚醒一次。
>
> 📌 **4 個時段的定位（v1.1 更新）**：改為批次清空制後，這 4 個時段的意義是「一天 4 次補發檢查點」——不論當下佇列累積了 1 支還是 10 支，該次觸發都會一口氣依序全部發完，而不是分散到多個時段慢慢發。

---

## 4. GitHub Repo 資料結構設計

### 4-1. `queue/pending.json`

```json
{
  "playlist_id": "PLxxxxxxxxxxxxxxxxx",
  "last_synced": "2026-09-16T06:00:00+08:00",
  "items": [
    {
      "video_id": "abc12345678",
      "title": "SIT Storage All-Stress Toolkit 解析",
      "playlist_item_id": "UExxxxxxxxxxxxxx",
      "position": 0,
      "added_at": "2026-09-10T20:00:00+08:00"
    }
  ]
}
```

### 4-2. `queue/published.json`

```json
{
  "history": [
    {
      "video_id": "xyz98765432",
      "title": "把踩過的坑寫進文件裡",
      "published_at": "2026-09-16T06:02:13+08:00",
      "trigger_slot": "06:00",
      "run_id": "github-actions-run-id"
    }
  ]
}
```

> 此檔案作為「防重複發布」的第二道防線：即使播放清單移動因故失敗，腳本也能靠 `video_id` 是否已存在於此清單，避免同一支影片被重複呼叫 `videos.update`。

---

## 5. YouTube Data API v3 — OAuth 2.0 設定步驟

1. 前往 [Google Cloud Console](https://console.cloud.google.com/)，建立新專案（或使用現有專案）
2. 左側選單「API 和服務」→「程式庫」，搜尋並啟用 **YouTube Data API v3**
3. 左側選單「API 和服務」→「OAuth 同意畫面」
   - User Type 選擇 **External**
   - 填寫應用程式名稱（如 `ChouAP-YT-AutoPublish`）、支援電子郵件
   - Scopes 新增：`https://www.googleapis.com/auth/youtube`
   - 測試使用者：先加入您自己的 Google 帳號
4. **【關鍵】** 回到 OAuth 同意畫面總覽頁，點擊「**發布應用程式（Publish App）**」，將狀態從 Testing 改為 **In Production**（見第 0-2 節說明，避免 7 天 Token 過期）
5. 左側選單「API 和服務」→「憑證」→「建立憑證」→「OAuth 用戶端 ID」
   - 應用程式類型選擇「**電腦版應用程式（Desktop app）**」（適合本機一次性授權取得 Refresh Token 的情境）
   - 下載憑證 JSON（`client_secret.json`）
6. 在本機（或任一能開瀏覽器的環境）執行一次性授權流程：
   - 使用 Google 官方 `google-auth-oauthlib` 套件的 `InstalledAppFlow`，帶入 scope `https://www.googleapis.com/auth/youtube`
   - 瀏覽器會跳出「未驗證應用程式」警示 → 點「進階」→「前往 ChouAP-YT-AutoPublish（不安全）」→ 允許權限
   - 流程完成後取得 `refresh_token`
7. 將以下三項存入 **GitHub Secrets**（Repo → Settings → Secrets and variables → Actions）：
   - `YT_CLIENT_ID`
   - `YT_CLIENT_SECRET`
   - `YT_REFRESH_TOKEN`

> API 配額提醒：`videos.update` 每次呼叫消耗 50 units，`playlistItems.insert`/`delete` 各 50 units，`playlistItems.list` 1 unit，單支影片完整發布約消耗 150 units。**改為批次清空制後，配額用量會隨佇列累積量變動**——例如佇列一次累積 10 支未發，該次觸發會一口氣消耗約 1,500 units。以 Google 預設每日 10,000 units 配額估算，單次批次上限約可處理 60+ 支仍在安全範圍內，正常使用情境無需申請提高額度；但若佇列曾長期未清、一次累積數十支以上，建議留意當日配額是否足夠。

---

## 6. LINE Messaging API — 官方帳號申請步驟

> LINE Notify 已於 2025/3/31 全面停止服務，官方帳號與網頁已於同年 5/12 關閉，故改用官方建議的替代方案 LINE Messaging API。個人用量（每日最多 2 則通知）遠低於免費額度每月 200 則，免付費。

1. 前往 [LINE Developers Console](https://developers.line.biz/console/)，用個人 LINE 帳號登入
2. 建立一個 **Provider**（提供者，可視為專案分類，取名如 `ChouAP-Cloud`）
3. 在該 Provider 底下建立一個 **Messaging API Channel**（頻道）
   - 填寫頻道名稱、說明、分類（個人用途）
4. 進入該 Channel 設定頁 →「Messaging API」分頁：
   - 記下 **Channel Access Token（長期）**：點擊「Issue」產生，這是呼叫推播 API 要用的憑證
   - 記下該頻道的 **QR Code / Bot ID**
5. 用您自己的手機 LINE 掃描 QR Code，將此官方帳號加為好友
6. **簡化技巧**：因為只有您自己一位好友，推播 API 可直接使用 **Broadcast（廣播）端點**（`POST /v2/bot/message/broadcast`），不需要另外架設 Webhook 伺服器去取得您個人的 User ID，大幅簡化實作
7. 將以下存入 **GitHub Secrets**：
   - `LINE_CHANNEL_ACCESS_TOKEN`

---

## 7. 主程式邏輯（供 Claude Code 實作參考的虛擬碼）

```
主流程 main():
    載入設定（GitHub Secrets 環境變數）
    觸發時段 = 判斷目前時間對應 06:00/15:00/20:00/01:00 哪一時段（記錄於 Log，供追蹤）

    成功清單 = []
    失敗清單 = []

    嘗試:
        YT_Auth() → 取得有效 access token
        pending_list = YouTube.playlistItems.list(待發布播放清單ID)  # 依 position 排序

        如果 pending_list 為空:
            LINE.broadcast("⚠️ [ChouAP.Cloud] 佇列已空，本次無片可發，請補片")
            寫入 Log(等級=INFO, 內容="佇列為空，跳過本次發布")
            結束

        # ---- 批次迴圈：依序處理佇列中「所有」影片，直到跑完整個清單 ----
        對於 pending_list 中每一個 item（依 position 由小到大）:

            本地防重複檢查: 若 item.video_id 已存在 published.json
                → 記錄(等級=WARN, 內容="偵測到重複項目，跳過")
                → continue（處理下一支）

            嘗試:
                YouTube.videos.update(item.video_id, privacyStatus="public")
                YouTube.playlistItems.insert(已發布播放清單ID, item.video_id)
                YouTube.playlistItems.delete(item.playlist_item_id)  # 從待發布清單移除

                更新 pending.json（移除該項）
                更新 published.json（新增紀錄，含 trigger_slot、時間戳）

                成功清單.append(item.title)
                寫入 Log(等級=PASS, 內容=f"成功發布：{item.title}")

            例外處理（單支失敗，不中斷整批）:
                失敗清單.append(item.title)
                寫入 Log(等級=ERROR, 內容=f"發布失敗：{item.title}，原因={錯誤訊息}")
                continue（繼續處理下一支，不影響其餘佇列）

        # ---- 迴圈結束，彙總通知 ----
        通知文字 = f"🎉 [ChouAP.Cloud] 本次共發布 {len(成功清單)} 支：\n" + "\n".join(成功清單)
        如果 失敗清單 不為空:
            通知文字 += f"\n⚠️ {len(失敗清單)} 支失敗，詳見 Log：\n" + "\n".join(失敗清單)

        LINE.broadcast(通知文字)

        git commit + push（佇列狀態 + Log 檔）

    例外處理（整體層級，如認證失敗、網路中斷）:
        寫入 Log(等級=CRIT，內容=錯誤詳情)
        LINE.broadcast("🔴 [ChouAP.Cloud] 本次執行發生嚴重錯誤，未完成發布，請檢查 Log")
```

---

## 8. Log 設計規範

沿用既有腳本風格慣例：

| 規範項目 | 內容 |
|---|---|
| 雙格式輸出 | CLI 純文字 Log（GitHub Actions 執行紀錄自動保留）+ HTML 折疊式 Log（commit 回 Repo 的 `logs/` 資料夾）|
| 訊息分級 | 六級制：`INFO` / `PASS` / `WARN` / `FAIL` / `CRIT` / `ERROR`（ERROR 專用於「真實故障但可續行」，其餘提醒性質一律歸類 WARN，不誤標 ERROR）|
| HTML 呈現 | 每個時段的執行紀錄以摺疊區塊（`<details>`）呈現，展開後可看該次判斷過程、API 呼叫結果 |
| 時間戳記 | 每筆 Log 記錄含完整日期時間戳（含觸發時段標籤，如 `[06:00 SLOT]`）|
| 使用者可見訊息語言 | 一律英文（即使程式內部註解或設計文件為中文/日文，對外顯示訊息維持英文，減少多語系混淆問題）|

---

## 9. 部署前檢查清單

- [ ] YouTube 端已建立「待發布」與「已發布」兩個播放清單，並記下其 Playlist ID
- [ ] 確認往後上傳影片時，凡要進佇列的一律先設為 **Unlisted**
- [ ] Google Cloud OAuth 同意畫面已切換為 **In Production**
- [ ] 已用 Production 狀態重新產生一次 Refresh Token（非沿用 Testing 期間舊 Token）
- [ ] LINE Messaging API 官方帳號已建立，並已用個人 LINE 加為好友
- [ ] GitHub Repo 已設為 **Private**
- [ ] GitHub Secrets 已設定：`YT_CLIENT_ID`、`YT_CLIENT_SECRET`、`YT_REFRESH_TOKEN`、`LINE_CHANNEL_ACCESS_TOKEN`
- [ ] `.github/workflows/publish.yml` 的 4 組 cron 時間已依第 3 節對照表設定（UTC）
- [ ] 首次執行建議手動觸發（`workflow_dispatch`）驗證全流程，確認無誤後才交給排程自動跑

---

## 附錄：程式撰寫風格規範（供 Claude Code 實作時遵循）

以下為 Albert.Chou Script Style 既定慣例，實作 `scripts/publish.py` 與 workflow 檔案時套用：

- 腳本內部註解一律使用**日文**（含讀音/振り仮名），但所有對使用者顯示的訊息一律**英文**
- 所有路徑採用**相對路徑**設計，避免寫死絕對路徑
- 顏色代碼使用 `\033` ANSI escape 格式，透過變數預先定義，確保各種終端機環境相容
- 訊息分級固定六級制：`INFO/PASS/WARN/FAIL/CRIT/ERROR`，各級對應固定顏色
- 程式碼分段以明顯分隔線（`# ==========`）搭配段落小標題，強化可讀性，方便團隊維運與交接
- 主程式（`publish.py`）與其他輔助功能（如 Log 產生、API 封裝）應分離為獨立模組/函式
- 加入版本歷史區塊，記錄每次修改的版本號、日期、修改內容、設計者
- 新增程式碼須套用既有的檢查規則，不因「新寫的」而略過
- OS/環境版本顯示採**自動偵測**方式取得實際發行版資訊，不寫死字串

---

*本規格書為定案版本，交付 Claude Code session 進行實作。實作過程如遇規格未涵蓋的細節，請依此文件精神延伸判斷，或回來確認。*
