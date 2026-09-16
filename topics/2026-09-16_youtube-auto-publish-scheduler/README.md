# YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃

> 自動化工具／已完成端對端驗證：用 GitHub Actions 排程驅動「待發布」播放清單批次轉為公開，並用 LINE Messaging API 回報結果

| 項目 | 內容 |
|---|---|
| 日期 | 2026-09-16 |
| 分類 | 自動化工具 |
| 狀態 | 草稿 |
| YouTube | |
| NotebookLM | |

## 背景

本機電腦常態關機，無法用 Windows 工作排程器做「影片排程延遲公開」這件事，因此規劃借用 GitHub Actions 的雲端排程能力（cron，本機電腦免開機）取代。核心構想：新影片上傳時先設為「未列出（Unlisted）」放進 YouTube 的「待發布」播放清單，排入 position 順序，之後由排程程式在固定時段呼叫 YouTube Data API v3 把 `privacyStatus` 改成 `public`，達到「排程延遲公開」的效果。

這篇是技術規格紀錄（專案代號 `ChouAP.Cloud - YT-AutoPublish`，規格書 v1.2）：架構、資料結構、API 設定步驟、主程式虛擬碼寫在這裡，**實作程式碼放在另一個獨立 repo**（依本 repo `CLAUDE.md` 的內容規範，content-hub 只放 Markdown、不放程式碼／CI 工程邏輯，`.github/workflows/` 也僅限儲存庫生命週期的 LINE 事件通知，不得放內容生成或爬蟲等工程邏輯）——[`yt-auto-publish`](https://github.com/AlbertChou20250706/yt-auto-publish)（Private）。v1.0 骨架實作＋Google Cloud／LINE 全套設定＋GitHub Secrets 填入已全部完成，並用 2 支真實影片（長影音＋短影音）跑過一次 `workflow_dispatch` 端對端驗證：`videos.update` 正確把 Unlisted 轉為 Public、影片正確從「待發布佇列」移至「已發布紀錄」、LINE 收到彙總通知、`queue/` 與 `logs/` 正確 commit 回 repo。完整虛擬碼、資料結構、API 申請步驟見 [`notebooklm_sources/YOUTUBE_AUTO_PUBLISH_SPEC.md`](notebooklm_sources/YOUTUBE_AUTO_PUBLISH_SPEC.md)（使用者原始規格書全文）；repo 內另有 [`CLAUDE.md`](https://github.com/AlbertChou20250706/yt-auto-publish/blob/main/CLAUDE.md) 記錄 Albert.Chou Script Style 規範供未來維護參考。

## 做了什麼（規劃中的架構）

六段流程：**讀取佇列 → 判斷是否有片 → 批次逐支處理 → 更新狀態檔 → LINE 彙總通知 → commit 回 repo**。

1. **觸發層**：GitHub Actions `schedule`（cron），台灣時間每日 06:00 / 15:00 / 20:00 / 01:00 各觸發一次（UTC 對照見規格書第 3 節），另保留 `workflow_dispatch` 供手動補跑測試
2. **佇列讀取層**：呼叫 `playlistItems.list` 重新核對「待發布」播放清單目前實際內容，依 `position` 排序，同時比對本地 `queue/pending.json`
3. **批次處理層**：佇列若為空，直接 LINE 通知「本次無片可發」結束；若有片，**依序處理佇列中所有影片**（不是只發 1 支），每支執行：
   - `videos.update`：`privacyStatus` → `public`（冪等操作，見下方「關鍵發現」）
   - `playlistItems.insert` 加入「已發布」清單、`playlistItems.delete` 從「待發布」移除
   - 更新 `pending.json` / `published.json` 兩份狀態檔
   - 單支失敗記錄 `ERROR` 並跳過，**不中斷整批**，繼續處理下一支
4. **通知層**：LINE Messaging API `broadcast` 端點，彙總本次「共發布 N 支」＋逐支標題清單，若有失敗項目一併附註
5. **Log 層**：CLI 純文字 + HTML 折疊式雙格式，六級訊息分級（`INFO/PASS/WARN/FAIL/CRIT/ERROR`）
6. **版控層**：每次觸發結束後 `git commit + push` 佇列狀態與 Log 檔回 repo，作為歷史紀錄

## 關鍵發現／重點

- **改「單支發布」為「批次清空佇列」是 v1.1 的關鍵轉向**：一開始設計是每次觸發最多發 1 支，後來改為每次觸發把當下佇列**全部**依序發完。4 個時段因此變成「一天 4 次補發檢查點」的角色，不是分散發布的節流閥。
- **舊影片（已 public）與新影片（unlisted）可混放同一佇列**（v1.2 釐清）：`videos.update(privacyStatus="public")` 是冪等操作，對已公開的舊影片呼叫不會出錯、只是沒有實質效果，腳本不需要額外判斷「這支是不是已經 public」才決定要不要處理，統一呼叫即可，邏輯更單純。
- **YouTube OAuth 一定要先切到「In Production」再產生 Refresh Token**：停留在 Testing 狀態的 Refresh Token 會在 7 天後自動失效（`invalid_grant`），對無人值守的 GitHub Actions 是致命問題（靜默斷線，需人工重新授權）。若順序顛倒（先產生 Token 才切 Production），該 Token 仍可能帶著 Testing 期間的限制，必須重新走一次授權。
- **`videos.update` 搭配 `part=status` 時要先 `videos.list` 讀出完整 `status` 物件再整包送回**：只改 `privacyStatus` 單一欄位送出，可能意外把 `publishAt`、`selfDeclaredMadeForKids` 等未指定欄位重置為預設值。
- **LINE Notify 已於 2025/3/31 停用**，改用官方建議替代方案 LINE Messaging API；因為只有自己一位好友，可直接用 **Broadcast 端點**，不需要架 Webhook 伺服器去取得個人 User ID，大幅簡化實作。
- **雙重防重複機制**：播放清單本身的移動（待發布→已發布）是第一道防線，本地 `published.json` 依 `video_id` 比對是第二道防線——即使播放清單移動因故失敗，也不會重複呼叫 `videos.update`。
- **批次處理採「單支失敗不中斷整批」設計**：API 額度用盡、影片已被刪除等單支錯誤只記錄 `ERROR` 並跳過，確保單一壞資料不會卡住整條佇列。
- **API 配額隨佇列累積量變動**：單支完整發布約消耗 150 units（`videos.update` 50 + `playlistItems.insert` 50 + `playlistItems.delete` 50 + `list` 1），以每日預設 10,000 units 估算，單次批次上限約可處理 60+ 支仍在安全範圍；若佇列曾長期未清、一次累積數十支以上，需留意當日配額。
- **私有 repo 連續 60 天無 commit 會被 GitHub 自動停用排程**：本系統每次觸發都會 commit 佇列狀態與 Log，正常使用不會觸發此限制，但佇列長期空著超過 60 天需手動喚醒一次。
- **自動化程式碼與內容紀錄要分兩個 repo**：GitHub Actions workflow、`scripts/publish.py`、佇列 JSON 屬於工程檔案，依 `CLAUDE.md` 規範不能放進 content-hub；content-hub 只負責事後把「怎麼設計」寫成技術紀錄。

### 部署實測踩到的坑（規格書 v1.2 沒寫到的部分）

- **頻道既有的「待發布」播放清單不能直接沿用**：實測時發現頻道原本就有一個公開播放清單叫「待發布」，裡面已經放了 44 支**早就公開**的影片（純粹拿來做內容分類用，不是真的排程佇列）。若直接接上自動化，第一次觸發就會把這 44 支全部搬進「已發布」，打亂原有分類——解法是另外新建兩個名稱明顯區隔的專用播放清單（「YT-AutoPublish 待發布佇列」/「YT-AutoPublish 已發布紀錄」），只給自動化內部使用，跟頻道原有的內容分類播放清單完全脫鉤，一支影片本來就能同時存在多個播放清單。
- **佇列播放清單本身的瀏覽權限也要設 Private**：如果自動化的「待發布佇列」播放清單本身是公開的，即使裡面的影片是 Unlisted，只要有人找到這個播放清單連結還是能點進去看，等於還沒正式發布就先被看光，失去排程延遲公開的意義。
- **Google 近期把 OAuth 同意畫面改版成「Google Auth Platform」，切 Production 前多了新的必填欄位**：規格書 v1.2 只提到要填 App 名稱和 email，但實測發現「發布應用程式」按鈕會被擋下，額外要求「品牌」頁面填妥**應用程式首頁網址**與**隱私權政策網站**這兩個網址，並把它們的網域加進「授權網域」清單。因為這支工具永遠只有自己一個測試使用者、不會走 Google 正式驗證，這兩個網址內容不需要多正式（可沿用既有的作品集網站當首頁，隱私權政策沿用手上其他個人工具現成的說明頁面即可），登記授權網域這步也不需要走 Google Search Console 的擁有權驗證。
- **LINE Channel Access Token 直接沿用既有的「ChouAP.Cloud」Bot**：不需要另外申請新的 Provider／Channel，跟 `ai-stock-weekly-report-bot`、`stock-committee-bot`、`quality-picks-bot` 共用同一組長期 Token；**切記不要點「Reissue」重新核發**，那會讓舊 Token 立刻失效，同時弄壞其他三個已經在用這組 Token 的自動化。
- **手動 Re-run 一個「已經自己 commit+push 過」的 workflow 執行紀錄會失敗**：GitHub 的「Re-run jobs」會用當初觸發那個時間點的舊 commit 去跑，但 `main` 分支其實已經被那次執行自己的 commit 推進過了，導致重跑到最後 `git push` 時因為版本落後被拒絕（non-fast-forward）。這不是程式邏輯壞掉（`Run publish script` 那步依然正常跑完），純粹是「重跑自我寫回 repo 的 workflow」這個操作方式本身的已知副作用。解法：workflow 的 commit+push 步驟改成先 `git fetch` + `git rebase origin/main` 再 push；日常若要重測，也建議一律用「Run workflow」觸發全新執行，不要對舊 run 按 Re-run。

## 排程總覽（台灣時間 → UTC cron）

| 台灣時間 | 對應 UTC 時間 | cron 表達式 | 定位 |
|---|---|---|---|
| 06:00 | 前一日 22:00 | `0 22 * * *` | 補發檢查點 |
| 15:00 | 當日 07:00 | `0 7 * * *` | 補發檢查點 |
| 20:00 | 當日 12:00 | `0 12 * * *` | 補發檢查點 |
| 01:00 | 前一日 17:00 | `0 17 * * *` | 補發檢查點 |

> GitHub Actions 的 cron 排程不保證準時，系統忙碌時可能延遲數分鐘到數十分鐘，設計上接受此誤差，不追求秒級精準。

## 行動項（部署前檢查清單）

- [x] 另開獨立自動化 repo（[`yt-auto-publish`](https://github.com/AlbertChou20250706/yt-auto-publish)，Private），完成 `scripts/publish.py` + 4 個輔助模組 + `.github/workflows/publish.yml` + `queue/pending.json` / `queue/published.json` 骨架實作
- [x] `.github/workflows/publish.yml` 的 4 組 cron 時間依第 3 節對照表設定完成（UTC）
- [x] YouTube 端建立「YT-AutoPublish 待發布佇列」與「YT-AutoPublish 已發布紀錄」兩個專用播放清單（刻意不沿用頻道原有、內容已公開的「待發布」/「已發布」分類清單，避免搞混）
- [x] 確認往後上傳影片凡要進佇列一律先設為 **Unlisted**
- [x] Google Cloud OAuth 同意畫面切換為 **In Production**（須在產生 Refresh Token *之前* 完成）
- [x] 用 Production 狀態重新產生一次 Refresh Token（`yt-auto-publish` repo 內 `scripts/get_refresh_token.py` 本機執行取得）
- [x] LINE Messaging API：沿用既有「ChouAP.Cloud」Bot（跟 `ai-stock-weekly-report-bot` 等共用同一組 Channel Access Token），不需另外申請
- [x] `yt-auto-publish` repo 設定 GitHub Secrets：`YT_CLIENT_ID`、`YT_CLIENT_SECRET`、`YT_REFRESH_TOKEN`、`LINE_CHANNEL_ACCESS_TOKEN`、`YT_PENDING_PLAYLIST_ID`、`YT_PUBLISHED_PLAYLIST_ID`
- [x] 首次執行以 `workflow_dispatch` 手動觸發驗證全流程：用 2 支真實影片（長影音＋短影音）實測成功，LINE 通知、播放清單搬移、privacyStatus 轉換皆正確
- [x] 補強 `git push` 穩健性：commit 後先 `fetch` + `rebase origin/main` 再 push，避免排程重疊或重跑舊 run 時被 non-fast-forward 拒絕
- [ ] 交給排程自動跑穩定幾輪後，再回來這裡錄 YouTube 教學、回填發布狀態

## 延伸資源

- 完整技術規格書（v1.2 全文，含虛擬碼、資料結構、OAuth／LINE 申請步驟、程式風格規範）：[`notebooklm_sources/YOUTUBE_AUTO_PUBLISH_SPEC.md`](notebooklm_sources/YOUTUBE_AUTO_PUBLISH_SPEC.md)
- 相關文章／前作（分工模式參考：工程 repo + content-hub 文字紀錄分開）：本 repo [AI 股市週報自動化：GitHub Actions 觸發 + Claude 生成 + LINE 群組推播規劃](../2026-08-27_ai-stock-weekly-report-line-bot/README.md)
- 參考資料：YouTube Data API v3 官方文件、LINE Messaging API 官方文件、LINE Notify 服務停止公告
- 相關 repo：[`yt-auto-publish`](https://github.com/AlbertChou20250706/yt-auto-publish)（實作程式碼，Private）

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
