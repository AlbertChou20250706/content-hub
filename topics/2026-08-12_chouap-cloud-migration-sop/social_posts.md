# Social Posts｜ChouAP.Cloud 上雲 SOP：把專案從本機搬成 GitHub + Claude Code Cloud 工作流

> 影片發布後 24 小時內用，導流回 YouTube／content-hub。目前影片尚未上架，以下為草稿內容。

## Threads / X（短，≤ 280 字，含 1 個 hook）

專案搬上雲端後，不管在哪台裝置登入同帳號都能接續開發，不用再依賴某一台電腦有沒有開機。整理成八步驟 SOP，已在兩個性質完全不同的專案上實際驗證過。

🔗 https://github.com/albertchou20250706/content-hub/tree/main/topics/2026-08-12_chouap-cloud-migration-sop

## LinkedIn（技術向，可長一點，強調專業紀錄）

把專案從本機搬上雲端聽起來簡單，實際操作時很容易在幾個地方踩坑：GitHub repo 跟 Cloud environment 是兩件要分開設定的事、雲端主機預設工具鏈不是什麼都有、Setup script 只在全新 session 執行一次並會快照。

這篇整理出一套完整的八步驟 SOP——從本機 Git 初始化、GitHub repo 建立、Claude GitHub App 授權，到 Cloud environment 設定與驗證，並已在兩個性質不同的專案上實際驗證：一個需要額外安裝 PowerShell 工具鏈，另一個直接沿用預設環境就能運作。過程中也記錄了兩個實戰地雷，包括一個容易被忽略、會讓 Setup script 靜默失敗的寫法問題。

搬遷完成後，任何裝置登入同一個帳號都能直接接續工作，不再受限於單一台電腦。

🔗 https://github.com/albertchou20250706/content-hub/tree/main/topics/2026-08-12_chouap-cloud-migration-sop

## Facebook（較口語，適合帶個人心得）

最近把手上兩個專案都搬上雲端了，一個是需要額外裝 PowerShell 的工具鏈，一個是純 Markdown 內容庫，兩種完全不同的情境都跑過一輪，整理成一套 SOP。

中間有踩到一個蠻雷的地方：Setup script 如果用 `&&` 把一長串安裝指令串在一起，只要中間有一步失敗，後面全部不會執行，但錯誤又被吞掉讓系統誤以為裝好了——實際上工具鏈是空的。拆開成一行一行寫，問題就解決了。細節都寫在文章裡。

🔗 https://github.com/albertchou20250706/content-hub/tree/main/topics/2026-08-12_chouap-cloud-migration-sop

## Hashtags

#雲端開發 #GitHub #ClaudeCode #自動化 #SOP

## 發布檢查

- [ ] 連結已確認可正常開啟
- [ ] 已同步更新 `_log/publish_log.md`
- [ ] 已回填 content-hub 首頁 README 索引表格該筆狀態
