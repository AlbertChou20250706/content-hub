# Social Posts｜YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃

> 影片發布後 24 小時內用，導流回 YouTube／content-hub。目前影片尚未上架、自動化 repo 也還沒建立，以下為草稿內容。

## Threads / X（短，≤ 280 字，含 1 個 hook）

本機電腦常態關機，沒辦法用工作排程器做「YouTube 影片延遲公開」，改用 GitHub Actions 雲端排程 + LINE Messaging API 通知，一天 4 個時段自動把待發布佇列批次清空。正在整理架構規格，順手記錄下來。

🔗

## LinkedIn（技術向，可長一點，強調專業紀錄）

規劃一套「YouTube 影片排程自動發布系統」：用 GitHub Actions 的 cron 排程（台灣時間每日 06:00/15:00/20:00/01:00）驅動，讀取 YouTube「待發布」播放清單，依序把佇列中所有影片的 `privacyStatus` 改為 public，再搬到「已發布」清單、寫入雙重防重複紀錄，最後用 LINE Messaging API 彙總通知本次發布結果。

過程中確認了幾個容易踩雷的地方：YouTube OAuth 同意畫面沒切到「In Production」，Refresh Token 會在 7 天後靜默失效；`videos.update` 設定 `privacyStatus` 是冪等操作，因此新舊影片可以混放同一佇列，不需要額外判斷是否已公開。

這篇先記錄完整的技術規格，實作會另外開一個獨立 repo 進行。

🔗

## Facebook（較口語，適合帶個人心得）

最近在規劃一個小工具：讓 YouTube 影片可以「排程延遲公開」，不用自己顧電腦手動按發布，改用 GitHub Actions 雲端排程，一天固定幾個時段自動把待發布的影片清空發完，順便用 LINE 通知自己發了幾支。

過程中踩到一個坑：Google OAuth 沒切到正式版（Production），授權 Token 七天就會失效。先把規格寫下來，之後實作完再回來更新進度。

🔗

## Hashtags

#YouTube自動化 #GitHubActions #ClaudeCode #LINEBot #排程發布 #ChouAPCloud

## 發布檢查

- [ ] 連結已確認可正常開啟
- [ ] 已同步更新 `_log/publish_log.md`
- [ ] 已回填 content-hub 首頁 README 索引表格該筆狀態
