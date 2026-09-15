# 經營策略筆記

> 這份檔案放經營規劃、流程草案這類**內部筆記**，跟 `topics/`／`daily-tools/` 底下公開發布的內容分開。
> 之後說「去 content-hub 找」，指的就是來 `_notes/` 這裡翻。

## 內容變現流程（草案，之後可調整）

1. **選材**：找尋合適、有價值的工具
2. **GitHub 紀錄**：上傳／整理進 GitHub（SEO 錨點，這部分就是目前 `content-hub` + `daily-tool-digest` 在做的事）
3. **內容轉製**：設計轉錄／腳本，把技術素材轉成可發布的內容
4. **YouTube 上架**
5. **作品本身也上 GitHub**：把實際的工具／程式碼作品公開 release
6. **防呆機制**：分享歸分享，但要設計防止他人直接複製貼上整包程式碼的機制（保護作品）
7. **流量變現**：靠 YouTube + GitHub 兩邊的流量／曝光累積
8. **販售作品**（之後再討論，還沒定案）

## 第 5～8 步實現方案（草案）

### 原則

- 作品要在 GitHub 上「有貢獻」，同時有價值的程式碼不能被整包複製流出
- 販售採**小額買斷制**：一次性付費取得使用權，不含後續更新／維護保證，但買家可在 Issue 提問
- 初期用**多 repo 分工**，公開與私有分開管理

### 1. GitHub 貢獻可見度

即使核心程式碼放私有 repo，README、文件更新、release note、回覆 issue 都算貢獻活動，一樣能維持 contribution graph 綠格子。不必為了保護程式碼犧牲曝光度，兩者不衝突。

### 2. ~~買斷制＋無後續服務但可提問（私有 repo + collaborator）~~ ⚠️ 已取代

> **已被下方「銷售平台決定：Microsoft Store」取代**，改成程式碼根本不對外開放、只透過 Store 發行編譯後的 App。保留原始討論作紀錄：

不做 license key 系統，改用：私有 repo ＋ 付款後加對方為 collaborator／read 權限。條款寫清楚：

- 一次性買斷＝取得原始碼使用權
- 不含更新維護保證（不承諾 SLA）
- 但買家可在 Issue 提問，屬於社群式支援而非有償服務

### 3. 多 repo 分工

延續現有帳號慣例（PXE、SPD_Flash、PTU_CPU_Verify 各自一個 repo），訂成規範：

- 一個工具＝一個「公開展示 repo」（放 README＋編譯好的 release，如 PyInstaller 打包的 .exe）
- 對應一個「私有原始碼 repo」（放完整 `.py` 原始碼）
- 兩邊用同名＋ `-src` 後綴之類的命名規則對應，方便管理

### 4. 網站需求 ⚠️ 定位已更新

> 見下方「銷售平台決定」章節——網站的角色改成「轉繼站」，不是最終交易平台。

- **初期不需要**獨立網站，先用既有 GitHub Pages 作品集（Albert-Pyahaw）撐住展示＋導流角色
- 等真的要做「付款＋授權管理」，優先接現成的 Gumroad／LemonSqueezy／Stripe Payment Link，不必自建整站
- 等規模夠大、需要自己的品牌落地頁時，再考慮自建

## 銷售平台決定：Microsoft Store（A 版，已拍板）

### 結論

- 有價值的工具**不開源**，只打包成 exe／MSIX，上架 **Microsoft Store** 銷售
- 免費小工具維持現狀，走 GitHub 開源 ＋ `daily-tools` 那條線，兩者不衝突
- 建議用 **Nuitka**（編譯成機器碼）打包，而非 PyInstaller（容易被反解回原始碼）

### 導流漏斗：網站＝轉繼站，不是終點

```
YouTube（示範教學）
    │
    ▼
自己網站／GitHub（作品介紹、SEO 錨點、信任背書）
    │
    ▼
Microsoft Store（實際下載／付費頁面）
```

自己的網站或 GitHub repo 不放交易邏輯，只負責「介紹＋導購」——把觀眾從影片接住，最後導去 Microsoft Store 完成下載／購買。真正的金流、授權、版本更新都交給 Store 處理，不用自己維護。

### 成本比較：A. Microsoft Store vs B. 自架網站／Gumroad 直接賣 exe

| 項目 | A. Microsoft Store | B. 自己網站／Gumroad 直接賣 |
|---|---|---|
| 開發者帳號 | **免費**（個人／公司皆已取消註冊費） | 視平台，Gumroad 免費、自架網站需主機費 |
| 程式碼簽章憑證 | **不需要**——通過認證後 Microsoft 用官方憑證自動重新簽章 | **需要**：OV 約 US$65～230／年，EV 約 US$250～580／年 |
| 平台抽成 | 非遊戲類約 85～95% 歸開發者（依用戶怎麼找到 App 而定） | Gumroad 等平台約抽 10% |
| SmartScreen 信任 | Store 自動處理，無警告 | 未簽章會被 SmartScreen 擋；簽章後仍需時間建立信譽（OV 較慢、EV 較快） |
| 上架審核 | 需通過 Store 認證（政策合規檢查） | 無審核，上架快 |
| 總成本 | **趨近於 $0**，主要是送審／改版的時間成本 | 每年固定憑證費（約 NT$7,000～18,000，依匯率）＋平台抽成 |

**結論：目標受眾是 Windows 使用者的前提下，走 Microsoft Store 幾乎零成本，比自己買憑證划算，所以選 A 版。**

### 上架後會不會「時間到就下架」

不會。Microsoft Store 沒有「上架滿多久自動下架」的機制，只有**違反政策**或**認證抽查不合格**時才會被要求修正或移除。真正會過期的是簽章憑證本身（現行規定最長 1 年，需年年續約），但只要簽名時有加 **timestamp**，就算憑證過期，已發布的舊版本 exe 依然會被系統信任，不影響既有用戶。

## 品牌名稱：AlbertBiahal（已拍板，2026-09-07 定案）

- 個人／產品品牌統一定名為 **AlbertBiahal**（`@Biahal` 已被他人占用，改用這個組合）
- 視覺識別沿用既有頭像（工具圖示：扳手＋螺絲起子），之後影片縮圖、封面、App icon 都套用同一組視覺
- **YouTube handle：`@AlbertBiahal`**（已拍板，確認未被使用）
- **Microsoft Store Publisher name：AlbertBiahal**（已同步修改，與 YouTube handle 一致）
- 待辦：網域是否可用，統一各平台顯示名稱

## 執行順序建議

不要一次把所有工具都上架，先挑一個小工具（如 `OT_Calc_Launcher`）當試點，走完一次「開發者帳號申請（Publisher name：AlbertBiahal）→打包→認證送審→上架→網站導流」完整流程，跑通後再套用到其他工具。

**技術路徑修正（2026-09-08）**：原規劃「用 Nuitka 編譯成 exe」是假設試點工具是 Python 寫的 GUI 程式，實際查看 `OT_Calc_Launcher` repo 後發現它是**純 HTML/CSS/JS 的靜態網頁工具**（離線可用），完全沒有 Python 程式碼，Nuitka 不適用。改用對應的正確路徑：

- **PWA Builder**（https://www.pwabuilder.com/，Microsoft 官方免費工具）：把符合 PWA 規格的網頁直接轉成 MSIX，不需要編譯器
- 前置需求：`OT_Calc_Launcher` repo 要先補上 `manifest.json` 與 service worker 這兩個 PWA 必要檔案，再啟用 **GitHub Pages** 把 `OT_Calculator.html` 架成一個可被 PWA Builder 掃描的網址
- 上傳到 Partner Center 送審、上架**不收費**（帳號免費、也不需要買簽章憑證，跟前面成本比較表的結論一致）
- 這個修正只適用於這個試點工具；其他工具如果實際是 Python／原生程式，屆時要重新確認技術棧，不能照搬 PWA Builder 這條路

### 進度追蹤

- [x] **Microsoft Store 開發者帳號申請** — 已完成（2026-09-06）。帳戶類型：個人；帳戶狀態：活動；Publisher name：**AlbertBiahal**（2026-09-07 由 Biahal 改定）；賣家 ID：95971780
  - 註冊過程中曾卡在 Partner Center「訪問受限」錯誤，原因是直接貼深連結（`/dashboard/registration`）繞過了正常流程；改從官方入口 https://developer.microsoft.com/en-us/microsoft-store/register/ → 點「開始使用」才順利完成，供之後其他 repo／帳號申請參考
- [x] **`OT_Calc_Launcher` repo 補上 PWA 必要檔案** — 已完成（2026-09-08）。新增 `manifest.json`、`sw.js`（service worker）、`icons/icon-192.png`、`icons/icon-512.png`，並在 `OT_Calculator.html` 加上 manifest／theme-color／icon 連結與 service worker 註冊
  - 過程中 Claude GitHub App 一開始沒有這個 repo 的推送權限，已在 https://github.com/apps/claude/installations/select_target 手動把 `OT_Calc_Launcher` 加進允許存取清單解決；之後其他 repo 需要推送權限時可比照辦理
- [x] **啟用 GitHub Pages** — 已完成（2026-09-08）。網址：https://albertchou20250706.github.io/OT_Calc_Launcher/OT_Calculator.html ；瀏覽器已能正常跳出安裝提示（App 名稱／圖示正確顯示）
- [x] **用 PWA Builder 產生 MSIX 安裝包** — 已完成（2026-09-08）。掃描結果 0 錯誤／3 警告／7 提示（警告與提示為加分項，非必要），已下載三個檔案：`OT Calculator.msixbundle`（送審用）、`OT Calculator.sideload.msix`（本機測試用）、`OT Calculator.classic.appxbundle`（備用）
- [x] **本機測試 sideload 版本** — 已完成（2026-09-08）。`install.ps1` 因公司電腦的 PowerShell 執行原則被組織政策鎖住，改用 `powershell -ExecutionPolicy Bypass -File install.ps1`（單次繞過，不改全域設定）成功安裝；App 以獨立視窗執行（非瀏覽器分頁），系統彈出「已從 Microsoft Store 安裝為應用程式」確認訊息，整條 PWA → MSIX → 安裝 技術路徑驗證成功
- [x] **Partner Center 提交流程走完大半** — 進行中（2026-09-08～09）：
  - **定价和可用性**：免費（USD，零售價 $0）
  - **属性**：類別選「生產率」；PublisherDisplayName／App name／Package ID 必須跟 Partner Center「产品标识」頁登記的值完全一致，否則套件驗證會失敗（詳見下方教訓）
  - **年龄分级**：IARC 問卷全部選「否」（不涉遊戲／社交／位置／購買等），結果為最低分級（3+／所有人）
  - **包**：已上傳 `OT Calculator - AlbertBiahal.msixbundle`，驗證通過（Validated），Device family 勾選 Windows 10/11 Desktop
  - **隱私政策關鍵教訓**：即使產品不收集任何個資、隱私問卷選「否」，只要套件宣告了 `runFullTrust` 等受限功能，Partner Center 仍會強制要求隱私政策 URL，屬性頁才會顯示「完成」。已建立 `privacy-policy.html` 並透過 GitHub Pages 提供：https://albertchou20250706.github.io/OT_Calc_Launcher/privacy-policy.html
  - **Store 一览**：完成英語(美國)版文案（說明、簡短描述、關鍵字 7 個、開發者名稱）＋ 1 張桌面截圖
  - **提交选项**：發布時機選「通過認證後立即發布」；`runFullTrust` 受限功能說明已填（PWA Builder 打包 PWA 為 Windows App 的標準做法，Microsoft 會在認證階段審核這份說明，不會卡在送出這關，見下方教訓）
- [x] **正式送出「提交进行认证」** — 已完成（2026-09-09）。狀態：正在認證（提交 ✓ → 預處理中 → 認證 → 發布），Microsoft 預估數小時至最多 3 個工作日完成，通過後依設定自動發布上架
  - **教訓**：Partner Center 側邊欄的「完成／未完成」徽章有時候不會即時反映真實存檔狀態（「属性」「提交选项」都遇過填完存了但徽章沒更新的狀況）。與其一直重新整理／截圖比對，**直接點「提交進行認證」讓系統做最終正式驗證**更有效率——如果真的有缺漏，系統送出時會給明確錯誤清單
- [x] **審核通過，正式上架** — 已完成（2026-09-13）。收到 Microsoft Partner Center 郵件通知「Your submission for your app OT Calculator - AlbertBiahal has been successfully processed」，最多 2 小時內於 Store 對外可見。試點工具從帳號申請到正式上架**完整跑通一次**，之後其他工具可直接沿用這套流程（先判斷技術棧是 PWA 還是原生程式，PWA 走 PWA Builder，原生程式另外評估打包方式）
  - Store 連結：https://apps.microsoft.com/detail/9NKGTSLTN8RN
- [x] **網站導流串接** — 已完成（2026-09-13）。`OT_Calc_Launcher` repo README 頂部加上 Microsoft Store 徽章／連結 ＋ GitHub Pages 線上體驗連結；`OT_Calculator.html` 頁面內也加一條小橫幅導去 Store 下載頁，讓透過網頁版進來的人也能發現桌面 App
  - **YouTube 導流**：這個工具目前還沒拍影片，等之後實際拍攝時，依「未來影片製作 SOP」章節的規則（B. 付費商品類型）處理——說明欄放 Store 連結，不放原始碼連結

## 試點結論：第一個工具完整跑通

`OT Calculator - AlbertBiahal` 是第一個走完「品牌定名 → 開發者帳號 → PWA 化 → 打包 → 送審 → 上架 → 導流」全流程的工具，可作為之後其他工具（PXE、SPD_Flash 等）上架的標準範本。下一個工具開始前，建議先確認該工具的技術棧（PWA vs. 原生程式），再套用對應的打包路線。

## 每日小工具素材（本機路徑）⚠️ 更正：已經是現成、運作中的系統（2026-09-15 更正）

> 這節原本寫「還沒開始、待擴充」是錯的——當時不知道這套系統已經存在。2026-09-15 實際打開 `daily-tool-digest` repo 才發現它從 2026-08-18 就開始每天跑，registry 裡已經累積 21 筆挑選紀錄。以下是更正後的正確狀態：

- 素材來源：`daily-tool-digest`（獨立自動化 repo）+ `content-hub/daily-tools/`（只收成品 Markdown），**已經是現成、每天在跑的系統**，不是待建置項目
- 架構：**GitHub Actions**（排程，非本機 Windows 工作排程器）→ Python 腳本用 **Google Service Account** 讀 `G:\我的雲端硬碟\產品上架(Product Release Pipeline)\Scrip_application`（解決了「cloud session 讀不到本機 G:\」這個問題，用服務帳號繞過，不是靠本機執行）→ 產出 manifest → 餵給 **headless `claude -p`**（帶入 `PROMPT.md` 定義的完整決策邏輯）→ 自主選材、寫 `source.md`、更新 registry、開週彙整 PR、LINE 通知
- 選材邏輯：兩層去重（registry 比對＋content-hub 關鍵字搜尋）、新品優先、選完一輪後 revisit 最久沒被選的、掃不到候選就誠實回報不捏造
- 完整設計文件在 `daily-tool-digest/DESIGN.md`（記錄了版本演進與踩過的坑：PowerShell 5.1 雙向編碼問題、headless CLI 下 Gmail MCP 不可用改用 LINE、`--max-turns` 取代 `--max-budget-usd` 等），之後要參考既有自動化模式，先查這份文件
- 這套系統的架構模式（獨立工程 repo＋PROMPT.md 驅動 headless claude -p＋registry 狀態追蹤＋LINE 通知）是後續「維護 Agent」等新自動化的標準範本，見下方「維護 Agent 藍圖」章節

### daily-tools 呈現方式

- **GitHub Pages**（沿用既有作品集站 Albert-Pyahaw）：做一個輕量「工具圖鑑」頁，每個工具一張卡片（縮圖＋一句話痛點＋連結），比純 Markdown 列表更有瀏覽感
- **YouTube Shorts**：60 秒內展示工具操作畫面，日更量產成本最低，適合「每天一支」的節奏
- GitHub／`daily-tools/` 本身仍當 SEO 錨點與 NotebookLM 音訊來源，這層不變

### 吸引觀眾注意力的原則

- 標題走**痛點／成果導向**，不走功能描述（例如「這支工具幫我省了 2 小時排查」優於「XX 監控工具介紹」）
- **前 3 秒先給結果**（Before/After 畫面），原理放後面
- **固定時間發布**做成儀式感，05:40 本身就可以變成品牌記憶點（例如「晨間工具站」）
- **縮圖／封面統一視覺模板**（同色系、同字體），累積出一致的品牌辨識度
- 短影音為主、長影音為輔，短影音負責導流
- **結尾留互動鉤子**（提問、邀請留言分享你在用的工具），拉高互動率

## 未來影片製作 SOP（草案，視覺設計之後再修）

### 通用（每支影片都要）

- 頻道頭像、縮圖模板套用同一組工具圖示＋色系，強化 AlbertBiahal 品牌辨識度
- 標題走痛點／成果導向，不寫功能描述
- 前 3 秒先給結果（Before/After 畫面），原理放後面
- 結尾留互動鉤子（提問、邀請留言分享你在用的工具）

### A. 免費小工具（daily-tools 那條線）

- 說明欄連到對應的 GitHub `daily-tools/` 條目（SEO 錨點）
- 固定時間發布（構想 05:40）維持儀式感

### B. 付費商品（未來 Microsoft Store 上架的工具，如試點的 `OT_Calc_Launcher`）

- 不放原始碼連結，只示範操作畫面（痛點→解法→成果）
- 說明欄放 Microsoft Store 下載／購買連結——導流漏斗（YouTube → 網站/GitHub 介紹 → Microsoft Store 購買）在這裡真正落地變現
- 可加一句「原始碼不公開，需要的功能歡迎透過 Store 頁面聯繫」，呼應防拷貝但不阻擋觀眾找到你的原則

## 營運追蹤：50+ 工具規模的資料管理（2026-09-14 決定）

**結論**：50+ 工具量級不需要真正的資料庫（SQL），Google Sheets／Notion 這類輕量表格工具就夠用，效能不是瓶頸。真正該優先做的是「可重複套用的 SOP」跟「營運追蹤表」，而不是資料庫系統本身。`content-hub` 這個 repo 保持只放 Markdown 公開內容的定位不變，不承擔追蹤表的角色（規範也禁止在這裡放工程邏輯／資料生成）。

### 追蹤表欄位設計（建議用 Google Sheets 或 Notion 建立，不放進 content-hub repo）

| 欄位 | 說明 |
|---|---|
| 工具名稱 | |
| 本機來源路徑 | `Scrip_application` 底下的資料夾名稱 |
| GitHub repo 連結 | |
| 技術棧 | PWA／Python／其他——**必須實際打開程式碼確認過才填**，不能用資料夾名稱或猜的（OT_Calc_Launcher 就曾經猜錯） |
| 目前階段 | 下拉選單，對應下方 SOP 的 8 個 Phase：未開始／技術棧確認中／GitHub 已推送／打包中／本機測試通過／已送審／審核中／已上架／內容已發布 |
| Publisher name／Package ID／Publisher ID | 上架後填，供之後核對一致性用（PWA Builder 每次重開表單容易跑掉，這幾個值要固定住） |
| Microsoft Store 連結 | |
| 隱私政策頁面連結 | |
| GitHub Pages 網址 | 若為 PWA |
| content-hub topic 資料夾連結 | |
| YouTube 長片／Shorts 連結 | |
| 上架日期 | |
| 備註／踩坑記錄 | 每個工具遇到的特殊狀況，累積下來就是自己的問題排除手冊 |

## 新工具上架 SOP（草稿，依 OT Calculator 試點經驗整理）

> 每一步都是這次試點實際走過、也實際踩過坑的流程，之後工具直接照這份勾，不用重新摸索。

### Phase 0：選材與技術棧確認
1. 從候選清單挑一個工具
2. **實際打開程式碼確認技術棧**——不要用 repo 描述或資料夾名稱猜（教訓來源：一開始以為 `OT_Calc_Launcher` 是 Python GUI，實際打開才發現是純 HTML/JS，整個技術路徑因此重新規劃過一次）
3. 依技術棧決定打包路線：純 HTML/JS 走 PWA Builder；Python GUI 需另外評估 Nuitka／PyInstaller（尚未驗證過，下一個 Python 工具會是第一次實測）

### Phase 1：GitHub 準備
4. 確認 repo 已存在、程式碼是最新版本
5. 確認 Claude GitHub App 有這個 repo 的 push 權限，沒有就去 https://github.com/apps/claude/installations/select_target 手動加

### Phase 2：PWA 化（僅適用純網頁工具）
6. 補 `manifest.json`——**name／short_name 要提前想好最終 Store 上架名稱**，避免後面 PublisherDisplayName 對不上（見 Phase 6 的教訓）
7. 補 service worker（`sw.js`）
8. 準備圖示（192×192、512×512）
9. 主 HTML 補上 manifest link／theme-color／service worker 註冊
10. 啟用 GitHub Pages
11. 瀏覽器打開確認會跳出安裝提示

### Phase 3：隱私政策（提前準備，不要等卡關才做）
12. 直接先寫一份 `privacy-policy.html` 放上 GitHub Pages，不管最後會不會被要求——PWA 打包出來的套件常帶 `runFullTrust` 等受限功能，Partner Center 屬性頁的隱私問卷不管選「是」或「否」，只要套件宣告了這類功能就會強制要求隱私政策 URL

### Phase 4：打包（PWA Builder 路線）
13. 用 PWA Builder 掃描 GitHub Pages 網址
14. **關鍵**：先去 Partner Center 該產品的「产品标识」頁查好 Package/Identity/Name、Package/Identity/Publisher（`CN=...`）、Publisher display name 三個值
15. 回 PWA Builder 的 Windows Package Options，**同一次表單裡**把這三個值＋App name（要跟 Partner Center 保留的 App 名稱完全一致）都填好才下載——**分開填、重新打開表單會重置成預設值**（`My Company Inc` 之類），這是這次踩最多次的坑
16. 下載確認產出 `.msixbundle`（送審用）／`.sideload.msix`（本機測試用）／`install.ps1`

### Phase 5：本機測試
17. 用 sideload 版＋`install.ps1` 測試安裝；若遇到 PowerShell 執行原則限制（公司電腦常見），用 `powershell -ExecutionPolicy Bypass -File install.ps1` 單次繞過，不用改全域設定
18. 確認 App 能以獨立視窗啟動（不是瀏覽器分頁）

### Phase 6：Partner Center 上架
19. 若尚未申請開發者帳號：務必從官方入口 https://developer.microsoft.com/en-us/microsoft-store/register/ 進入，**不要直接貼深連結**（否則會卡在「訪問受限」）
20. 新增產品 → MSIX 或 PWA 應用 → 保留 App 名稱（跟 PWA Builder 填的完全一致）
21. 定价和可用性：定價、市場（預設全球所有市場即可）
22. 属性：類別；隱私政策問卷**直接選「是」並提供 Phase 3 準備好的隱私政策網址**，不要選「否」浪費時間卡關
23. 年龄分级：IARC 問卷（純工具類選項通常一路選「否」即可，會得到最低分級）
24. 包：上傳 `.msixbundle`，勾選對應的 Device family（純桌面工具只勾 Windows 10/11 Desktop）
25. Store 一览：至少一種語言的文案（說明、簡短描述、關鍵字）＋至少 1 張截圖
26. 提交选项：發布時機選「通過認證後立即發布」；若有 `runFullTrust` 等受限功能，填寫使用理由說明（標準寫法：PWA Builder 官方打包工具的必要功能，僅用於以全信任 WebView2 容器執行、不存取檔案系統／網路／硬體）
27. **直接點「提交進行認證」**——Partner Center 側邊欄的完成／未完成徽章有時不會即時反映真實存檔狀態，與其一直重新整理比對，不如讓系統做最終正式驗證，有缺漏會給明確錯誤清單

### Phase 7：上架後
28. 等審核結果（數小時至最多 3 個工作日，會收到 Email 通知）
29. 收到通過通知後，確認 Store 連結可正常開啟

### Phase 8：內容與導流
30. GitHub repo README 補上 Store 徽章／連結、GitHub Pages 線上體驗連結
31. PWA 網頁本身加一條導去 Store 的橫幅
32. `content-hub` 建立 `topics/YYYY-MM-DD_slug/` 三件套（`README.md`／`youtube_metadata.md`／`social_posts.md`，從 `_templates/` 複製起手）
33. 回填根目錄 README 主題索引表、`_log/publish_log.md`
34. YouTube 說明欄放 Store 連結＋對應 GitHub repo 連結
35. 視情況剪一支 Shorts 導流（可另立獨立製作管線，不用是長片剪輯）
36. 社群貼文發佈（`social_posts.md` 草稿直接用）
37. 追蹤表狀態更新為「已上架／內容已發布」

## 維護 Agent 藍圖（2026-09-14／15）

- 決策圖：https://claude.ai/artifact/CMcoSXXzv7oDsCTwa46x9n（存放追蹤表欄位設計與新工具上架 SOP 的視覺化決策圖，之後可依實際狀況修正）
- **核心原則**：手動協作（on-demand session）永遠是預設；排程自動化只在真的划算時才加，而且加的時候是「一個共用 Routine」，不是「每個工具各一個」
- **升級門檻**：工具數 15-30+、開始有真實使用者 Issue／評論回饋、同類問題重複出現、自己巡查時間開始擠壓到做新工具——四項符合兩項以上再考慮開 Routine
- **安全邊界**：agent 能偵測、分類、草擬 PR；合併、發佈、Store 重新上架永遠是人工點頭，不自動跨過

### 已確認的設計標準（2026-09-15）

1. **三語 UI 是所有 scrip 工具的標準配置**：未來設計的工具都比照 `OT Calculator` 走繁體中文／英文／日文三語切換，目的是擴大上架後的市場價值（不是單一語系限定台灣市場）。「UI 英文＋註解日文」的雙語慣例**已排除**，那是誤植，不採用。
2. 銷售平台維持 Microsoft Store 定案（見前面「銷售平台決定」章節），Gumroad 只是核對筆記記錄有沒有記準，不是要重新考慮。

### 重大發現：`daily-tool-digest` 提供了現成的自動化範本

實際打開該 repo 才發現這套系統已經運作近一個月（見上方「每日小工具素材」章節更正），架構是：**獨立工程 repo ＋ GitHub Actions 排程 ＋ PROMPT.md 驅動 headless `claude -p` ＋ registry 狀態追蹤 ＋ LINE 通知**，且已經在 `DESIGN.md` 裡記錄了大量實戰踩坑經驗（PowerShell 編碼問題、headless 下 Gmail MCP 不可用、`--max-turns` 用法等）。

### 下一步：Issue 掃描維護 Agent（籌備中，尚未開工）

沿用 `daily-tool-digest` 已驗證的架構模式，對應關係：

| daily-tool-digest 的角色 | 維護 Agent 的對應設計 |
|---|---|
| 掃 Google Drive 找候選 | 掃所有已上架工具 repo 的 GitHub Issue（`gh issue list` 即可，不需要 Google API 那層複雜度） |
| `registry/picked-tools.json` | 追蹤「哪些 Issue 已處理過」 |
| PROMPT.md 驅動 headless claude -p | 同樣模式：讀 Issue → 判斷複雜度 → 簡單的草擬 PR／複雜的標記 |
| 週彙整 PR 進 content-hub | 改成**直接對該工具自己的 repo 開 PR**（程式碼修復，不進 content-hub） |
| LINE 通知 | 沿用同一支通知機制 |

**待確認的兩個開放問題**（比照 daily-tool-digest 當初「先問過 Albert 再定案」的作法）：
1. 新開一個獨立 repo（如 `tool-maintenance-digest`），還是掛在 `daily-tool-digest` 底下當第二個 workflow？（建議新開，因為「發現新素材」跟「維護已上架產品」職責不同）
2. 一開始掃全部工具 repo，還是先挑 1-2 個已上架的（目前只有 `OT_Calc_Launcher`）小範圍試跑？（建議先小範圍）
