# ChouAP.Cloud 上雲 SOP：把專案從本機搬成 GitHub + Claude Code Cloud 工作流

> 教學 SOP：不依賴單一台電腦開機，任何裝置登入同帳號都能接續開發

| 項目 | 內容 |
|---|---|
| 日期 | 2026-08-12 |
| 分類 | 教學 SOP |
| 狀態 | 草稿 |
| YouTube | |
| NotebookLM | |

## 背景

ChouAP.Cloud 是把本機或雲端硬碟同步路徑（如 Google Drive、OneDrive 同步資料夾）下的專案，搬成「GitHub repo + Claude Code Cloud environment」雲端工作流的品牌名稱。搬遷完成後，任何裝置登入同一個帳號都能直接接續工作，不再依賴單一台電腦是否開機、是否在身邊。

這套 SOP 已在兩個性質截然不同的專案上實際驗證過：

- **audio_transcribe**：PowerShell 工具鏈，雲端主機預設不含 PowerShell，需要靠 Setup script 額外安裝 pwsh
- **content-hub**（本 repo）：純 Markdown 內容庫，不需要任何額外工具，直接沿用 Default environment 就能運作

在動手之前，有四個核心觀念要先建立：

1. **GitHub repo 是單一事實來源**：Cloud session 是靠 clone 存取程式碼，不是直接讀本機資料夾。搬遷沒做完、程式碼還留在本機沒推上去，雲端就看不到。
2. **Cloud environment 不等於 repo**：這是兩件分開設定的東西，開 session 時兩個都要選對，不是選了 environment 就自動連到某個 repo。
3. **雲端主機預設環境是 Ubuntu 24.04**，內建 Python、Node、Go、Rust、Java、Ruby、PHP、Docker，但 PowerShell、.NET SDK 這類不在預設清單內的工具，要靠 Setup script 自己裝。
4. **Setup script 只在開全新 session 時執行一次，並且會做快照**：之後開新 session 會直接沿用快照、不重跑；但 resume 既有 session 同樣不會重跑 Setup script，所以工具鏈的變更只對「全新 session」生效。

## 做了什麼

0. 若專案原本放在雲端硬碟同步路徑下（例如 Google Drive 桌面同步資料夾），先**複製（不是剪下）**一份到本機非同步路徑。避免 Git 操作過程中產生的檔案變動被同步軟體同時偵測、互相干擾。
1. 準備三個骨架檔案：`.gitignore`、`README.md`、`CLAUDE.md`。`CLAUDE.md` 是給 Claude Code 之後接手這個 repo 時的工作指引，先寫好可以省掉後續每次重新交代背景。
2. 在本機非同步路徑下執行 `git init` → `git add .` → `git commit`，把專案變成一個本地 Git repo。
3. 到 GitHub 建立一個空的 **Private repo**，**不要**勾選 Initialize with README（避免遠端一開始就有本機沒有的 commit，造成後續 push 衝突）。
4. `git remote add origin` → `git branch -M main` → `git push -u origin main`，把本機 repo 推上 GitHub。
5. 安裝並授權 **Claude GitHub App** 存取這個 repo。建議選 **Only select repositories**，只授權要用到的 repo，範圍愈小愈安全。
6. 建立 **Cloud environment**：
   - Name：填專案名稱，方便日後在多個專案之間辨識
   - Network access：維持 **Trusted**
   - Environment variables：**留空**，因為目前沒有 secrets 保管機制，不能把 API Key 這類敏感資訊放在這裡
   - Setup script：依專案實際語言／工具需求填寫（細節與地雷見下方「關鍵發現」）
7. 開一個新 session 實際驗證：
   - 先確認需要的工具是否已經預裝好（例如執行 `pwsh --version` 確認 PowerShell 真的裝上了），不要讓 Claude Code 在任務當下才臨時安裝
   - 再實際跑一次完整流程，確認全程零錯誤，才算搬遷完成

## 關鍵發現／重點

- **GitHub repo 與 Cloud environment 是兩個獨立設定**，開 session 時要分別選對，別以為選了 environment 就自動綁定了某個 repo。
- **雲端主機的預設工具鏈已經很完整**（Python/Node/Go/Rust/Java/Ruby/PHP/Docker），但不是「什麼都有」——PowerShell、.NET SDK 這類工具需要額外用 Setup script 安裝，不要預設它一定存在。
- **Setup script 是「開全新 session 時跑一次 + 快照」的機制**，resume 舊 session 不會重跑。這代表如果要調整工具鏈安裝內容，必須驗證時故意開一個全新 session，而不是在原本的 session 裡改一改繼續用。
- **雷點 1**：commit 前忘記 `git add .`，會出現 `nothing added to commit but untracked files present`。看似小事，但第一次搬遷時很容易漏掉這一步，尤其是複製檔案到新路徑後忘記重新加入暫存區。
- **雷點 2（影響最大）**：Setup script 不要用 `&&` 把一長串安裝指令串在一起。實際踩過的狀況是 `apt update` 因為某個無關的 PPA 回應 403，導致用 `&&` 串起來的整條鏈直接中斷，後面所有安裝指令根本沒有執行——但因為外層又包了 `|| echo ... ; exit 0` 之類的容錯，錯誤被吞掉，Setup script 回報成功，讓系統誤以為環境已經裝好，實際上工具鏈是空的。正確做法：**每個安裝步驟拆成獨立的一行**，`apt update` 後面單獨接 `|| true`，讓某一步的失敗不會拖垮後面所有步驟，也不會被錯誤地吞成「成功」。

## 行動項（若有後續待辦）

- [ ] 錄製 YouTube 教學影片，實際示範 audio_transcribe 與 content-hub 兩個案例的搬遷過程
- [ ] 補充 Setup script 的實際範例內容（拆行寫法 vs. 地雷寫法的對照）
- [ ] 視情況追加第三個驗證案例，涵蓋更複雜的多語言工具鏈情境

## 延伸資源

- 相關 repo：本 repo（content-hub，本篇即為驗證案例之一）、audio_transcribe（PowerShell 工具鏈驗證案例）
- 相關文章／前作：無
- 參考資料：無

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
