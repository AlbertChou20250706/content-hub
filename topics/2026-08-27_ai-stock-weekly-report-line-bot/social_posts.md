# Social Posts｜AI 股市週報自動化：GitHub Actions 觸發 + Claude 生成 + LINE 群組推播規劃

> 影片發布後 24 小時內用，導流回 YouTube／content-hub。目前影片尚未上架、自動化本身也還在規劃階段，以下為草稿內容。

## Threads / X（短，≤ 280 字，含 1 個 hook）

想讓 AI 股市週報自己跑：GitHub Actions 排程觸發、Claude 生成內容、LINE Messaging API 推播到群組，全程不用顧著開機。正在整理架構規劃，順手記錄下來。

⚠️ 投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議

🔗 https://github.com/albertchou20250706/content-hub/tree/main/topics/2026-08-27_ai-stock-weekly-report-line-bot

## LinkedIn（技術向，可長一點，強調專業紀錄）

規劃一套「AI 股市週報自動化」：用 GitHub Actions 的排程與手動觸發（`schedule` + `workflow_dispatch`）驅動整條流程——抓股市資料、丟給 Claude API 生成週報摘要、再透過 LINE Messaging API 推播到指定群組。

過程中確認了一個容易踩雷的地方：LINE Notify 已於 2025 年 3 月停止服務，必須改用 LINE Messaging API 的 Bot channel 架構。另外也把「免責聲明必須固定內嵌在 AI 生成內容裡」定為硬性規則，而不是事後人工補加。

這篇先記錄完整的技術規劃，實作會另外開一個獨立 repo 進行。

⚠️ 投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議

🔗 https://github.com/albertchou20250706/content-hub/tree/main/topics/2026-08-27_ai-stock-weekly-report-line-bot

## Facebook（較口語，適合帶個人心得）

最近在規劃一個小工具：讓 AI 每週自動寫一篇股市週報，發到 LINE 群組，觸發方式借用 GitHub Actions 的排程功能，感覺有點像雲端上養了一個小助理。

過程中發現原本想用的 LINE Notify 已經停止服務了，得改用 Messaging API 的 Bot 架構。先把整體規劃寫下來，之後實作完再回來更新進度。

⚠️ 投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議

🔗 https://github.com/albertchou20250706/content-hub/tree/main/topics/2026-08-27_ai-stock-weekly-report-line-bot

## Hashtags

#AI股市週報 #GitHubActions #ClaudeCode #LINEBot #自動化 #投資理財

## 發布檢查

- [ ] 連結已確認可正常開啟
- [ ] 已同步更新 `_log/publish_log.md`
- [ ] 已回填 content-hub 首頁 README 索引表格該筆狀態
- [ ] 每則貼文皆含免責聲明「投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議」
