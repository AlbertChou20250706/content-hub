# YouTube Metadata｜免打包 Electron 也有桌面體驗：用純 HTML + Edge 啟動器打造 OT Calculator

## 標題（實際發布用，≤ 100 字元）

免打包 Electron 也有桌面體驗【純 HTML 配合 Edge 啟動器】

## 說明欄（Description）

本集剖析專為 SIT 與製造團隊打造的單檔離線加班時數試算工具 OT Calculator。內容拆解標準工時順延、晚餐扣除與四種申報進位演算法，解析防抖動除錯開關與 Edge 類原生啟動器架構，並從版本演進落差反思個人工具迭代與 vibe coding 實踐。

【✨ 章節 ✨】
0:00 開場
1:46 手算加班費，正式喊卡
2:34 加班到底幾點才起算？
3:31 75分鐘加班，算幾小時？
4:52 瘋狂連點按鈕，會怎樣？
6:11 零依賴，也有原生質感
7:26 功能超前，部署卻慢半拍

━━━━━━━━━━━━━━━━━━
📖 本集重點
- 解決手算痛點：製造業與 SIT 團隊常面臨午休扣除、晚餐起算門檻及進位規則因公司而異的困擾，OT Calculator 將規則參數化，輸入打卡時間即自動換算淨工時與申報小時數。
- 核心時間邏輯：預設 8 小時工時，跨 12:00 至 13:00 午休時由 standardEnd() 自動順延下班時間；加班時間嚴格從標準下班時間加上 0.5 小時晚餐時間後起算，避免算入吃飯時間。
- 申報換算機制：加班分鐘數需達最低門檻（預設 1 小時）才予採計，透過 roundClaimMins() 支援捨去到 1 小時、四捨五入到 1 小時、進位到 1 小時或四捨五入到 0.5 小時等四種進位方式。
- 防抖動保護設計：v1.1.1 內建 const GUARD = false 除錯開關，搭配 smallDrift 與 hugeJump 門檻過濾異常時間差，防止按鈕重複點擊造成畫面閃爍或數據錯亂。
- 輕量類桌面啟動：PowerShell 啟動器 OT_Calc_Launcher.ps1 偵測 Edge 瀏覽器並以 --app= 參數無框啟動，提供原生視窗體驗且無需打包 Electron。
- 單檔零依賴架構：採用純原生 vanilla JS 與 inline CSS 封裝於單一 HTML 檔案，完全在瀏覽器端本機離線運作，無資安外洩風險且易於內網隨身碟分發。
- 版本演進與實踐啟示：從 v1.0 演進至 v1.1.1 再到 v1.3.0（新增三語系、週看板、CSV 匯出與主題），揭示個人工具「功能領先、部署落後」的真實工程現象，為 vibe coding 提供最佳範例。

━━━━━━━━━━━━━━━━━━
【📚 延伸閱讀】
🔗 完整技術紀錄：https://github.com/AlbertChou20250706/content-hub/tree/main/topics/2026-09-05_ot-calculator-html-edge-launcher
🔗 想要桌面 App 版本？從 Microsoft Store 下載：https://apps.microsoft.com/detail/9NKGTSLTN8RN
🔗 作品集：https://albertchou20250706.github.io/Albert-Pyahaw/

【🔔 追蹤 Albert】
🎬 YouTube：https://www.youtube.com/@AlbertBiahal
💻 GitHub：https://github.com/AlbertChou20250706/OT_Calc_Launcher

━━━━━━━━━━━━━━━━━━
【🎬 本集製作】
監製・企畫・錄音｜Albert.Chou
轉錄・校對・字幕・影片｜AI 自動化工作流
（Whisper 語音轉錄 × Claude 校對整理 × NotebookLM 影片生成）

完整工作流與腳本已開源，歡迎交流。

OT Calculator, PWA, Microsoft Store, Windows App, 加班計算, SIT, 免打包

> ✅ 已同步（2026-09-13）：延伸閱讀的 Store／content-hub／GitHub repo 連結已實際更新進 YouTube 說明欄（已截圖確認）。結尾 hashtag 是否已從泛用標籤換成跟本集相關的內容、`#Shorts` 是否已移除，尚未實際核對，記得自行檢查一下 YouTube Studio 目前的最終內容，之後有空再回來把這份存檔更新成完全一致的版本。

## 時間戳章節（Chapters）

0:00 開場
1:46 手算加班費，正式喊卡
2:34 加班到底幾點才起算？
3:31 75分鐘加班，算幾小時？
4:52 瘋狂連點按鈕，會怎樣？
6:11 零依賴，也有原生質感
7:26 功能超前，部署卻慢半拍

## 標籤（Tags，逗號分隔，≤ 500 字元）

OT Calculator, 加班計算, PWA, Microsoft Store, Windows App, HTML, JavaScript, Edge, 免打包, SIT, 製造業, vibe coding

## 分類（Category）

Science & Technology

## 縮圖文字（Thumbnail Text，≤ 6 字為佳）

告別 Excel 手算加班

## 播放清單

## 卡片／結束畫面（Cards / End Screen）

- 卡片：
- 結束畫面推薦影片：

## SEO 關鍵字檢查

- [x] 標題含核心關鍵字
- [x] 說明欄前兩行含核心關鍵字
- [ ] 標籤涵蓋同義詞／相關搜尋詞（目前說明欄 hashtag 偏泛用，跟本集內容關聯度不高，可考慮換成更貼合 OT Calculator／PWA／Microsoft Store 主題的標籤）

---

# Shorts 版本

## 連結

https://www.youtube.com/shorts/ep86NV-DBjg

## 製作方式

走既有的 Shorts 製作管線 `G:\我的雲端硬碟\YouTube工作流\錄音工作流\Short_audio_transcription`，不是從長片剪出精華片段，是獨立產出的一支短影音。

## 標題

【直述型】單檔 HTML 打造離線 OT Calculator：打卡換算與防護實戰

## 說明欄（Description）

內容與長片版本完全相同（含延伸閱讀連結、追蹤資訊、本集製作標示），結尾 hashtag 為 `#Shorts #SIT #Linux #工程師工具`。

> 📝 備註：說明欄整段照搬長片文字，沒有針對 Shorts 觀眾精簡。Shorts 觀眾多半不會展開長文字，之後可考慮改成更短、更直接的版本（開頭鉤子 + 1-2 句重點 + 連結），提升導流轉換率。
