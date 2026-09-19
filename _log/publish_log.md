# Publish Log

> 每次發布（YouTube 上架、social post 發出）都在此追加一筆，作為 content-hub 的發布歷史紀錄。

| 日期 | 主題 | 動作 | 連結 | 備註 |
|---|---|---|---|---|
| 2026-08-11 | 三步驟搞定伺服器電壓量測：DSOX4024G 示波器接線與存檔實戰教學 | content-hub 主題頁建立並回填 | https://youtu.be/nFGo77bOH-w | YouTube 影片實際發布於 2026-08-04；本篇為 content-hub 首輪內容，反向回填 |
| 2026-08-12 | ChouAP.Cloud 上雲 SOP：把專案從本機搬成 GitHub + Claude Code Cloud 工作流 | content-hub 主題頁建立 | topics/2026-08-12_chouap-cloud-migration-sop/README.md | GitHub 文字紀錄先行發布；YouTube 影片與 social post 待錄製、待發 |
| 2026-08-13 | ChouAP.Cloud 上雲 SOP：把專案從本機搬成 GitHub + Claude Code Cloud 工作流 | YouTube 上架，主題索引與系列集數表回填 | https://www.youtube.com/watch?v=L16n_oDjXwQ | 狀態由草稿改為已發布；同步更新根目錄 README 與 series/claude-code-cloud-dev/README.md |
| 2026-08-27 | AI 股市週報自動化：GitHub Actions 觸發 + Claude 生成 + LINE 群組推播規劃 | content-hub 主題頁建立 | topics/2026-08-27_ai-stock-weekly-report-line-bot/README.md | GitHub 文字紀錄先行建立；目前為技術規劃階段，尚未開始實作，YouTube 影片與 social post 待發布 |
| 2026-09-16 | YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃 | content-hub 主題頁建立 | topics/2026-09-16_youtube-auto-publish-scheduler/README.md | 使用者提供技術規格書（v1.2）全文存入 notebooklm_sources；GitHub 文字紀錄先行建立，實作程式碼待另開獨立 repo，尚未開始工程實作，YouTube 影片與 social post 待發布 |
| 2026-09-16 | YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃 | 獨立自動化 repo 實作完成並端對端驗證成功 | https://github.com/AlbertChou20250706/yt-auto-publish | 新開 `yt-auto-publish`（Private）完成 v1.0 骨架實作；Google Cloud OAuth（含新版 Google Auth Platform 品牌/授權網域設定）、LINE Messaging API（沿用既有 ChouAP.Cloud Bot）、6 組 GitHub Secrets 全部設定完成；用 2 支真實影片（長影音＋短影音）跑 `workflow_dispatch` 實測成功，並補強 git push 穩健性（rebase 再 push，避免排程重疊/重跑衝突）；主題頁回填踩坑紀錄；YouTube 教學影片與 social post 仍待發布，狀態維持草稿 |
| 2026-09-17 | YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃 | 排程頻率調整（`yt-auto-publish` v1.1） | https://github.com/AlbertChou20250706/yt-auto-publish | 真實排程觸發（4 個時段）跑了一輪後，發現佇列為空的時段一樣會發 LINE 通知，一天最多耗掉 4 則免費額度，且與另外 3 個自動化共用同一組 Token／每月 200 則額度；因批次清空設計本身已保證不漏發，改為每日僅 02:00 觸發一次，降低通知消耗；主題頁排程總覽表與踩坑紀錄同步更新，舊 4 時段對照表收合保留供歷史查閱 |
| 2026-09-19 | YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃 | 發布節奏調整（`yt-auto-publish` v1.2） | https://github.com/AlbertChou20250706/yt-auto-publish | 「批次清空佇列」改回「每次觸發最多成功發布 1 支」（`config.MAX_PUBLISHES_PER_RUN`），佇列累積再多也是一天發 1 支、依序留到之後幾天；失敗項目不計入上限、繼續嘗試下一支；LINE 通知新增「還剩幾支排隊中」附註；已用模擬 10 支佇列的隔離測試驗證（1 支成功、9 支留待下次）；主題頁「做了什麼」「關鍵發現」「踩坑紀錄」「行動項」同步更新 |
