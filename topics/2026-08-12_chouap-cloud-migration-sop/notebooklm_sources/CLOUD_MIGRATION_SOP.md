# 專案搬上雲端 SOP — 可重複套用流程

> 把一個本機（或雲端硬碟同步路徑下）的專案資料夾，搬成「GitHub repo + Claude Code Cloud environment」的雲端工作流。搬完後任何裝置登入同帳號都能接續，不再依賴單一台電腦開機。
>
> 本文件由 `audio_transcribe` 專案的實際搬遷流程固化而成，已完整驗證可行。

**設計者：** Albert Chou
**文件版本：** v1.0.0（初版，涵蓋八步驟搬遷流程、CLAUDE.md/README.md/.gitignore 範本、兩個實戰雷點）

---

## 適用情境

- 專案目前放在本機或 Google 雲端硬碟同步路徑下，只能在特定電腦上用 Local session 操作
- 希望改成「Cloud 為主、Local 桌面為輔」，跨裝置接續
- 想讓 Claude 每次開 session 都能直接讀寫這批程式碼，不依賴任何一台電腦保持開機

## 核心觀念（先搞懂，後面才不會卡）

1. **GitHub repo = 單一事實來源**。Cloud session 是在 Anthropic 雲端一台獨立、乾淨的 Ubuntu 主機上跑，透過 GitHub clone 你的程式碼進去，不是直接讀你電腦的資料夾。所以程式碼一定要先在 GitHub 上。
2. **Cloud environment ≠ repo**。兩者常同名容易混淆：environment 是「執行環境」（要裝什麼工具），repo 是「程式碼本身」。開 session 時兩個都要選。
3. **雲端主機預設是 Ubuntu 24.04，內建工具有限**。Python、Node、Go、Rust、Java、Ruby、PHP、Docker 這些預裝好了；但 PowerShell、.NET SDK 等不在預設清單，要靠 Setup script 自己裝。
4. **Setup script 只在「開全新 session」時跑，且會快照**。第一次跑完會存成快照，之後開新 session 直接沿用、不重裝。改了 script 內容或約七天後快照會失效重建。Resume 既有 session 不會重跑。

---

## 搬遷流程（八步）

### Step 0 — 把資料夾複製到非雲端同步路徑

如果專案在 `G:\我的雲端硬碟\...` 這類雲端同步路徑下，**先複製（不要剪下）**整份到本機路徑，例如 `C:\Projects\<專案名>`。

原因：Git 的 `.git` 版控資料庫檔案變動頻繁，跟 Google Drive 這類同步軟體放一起容易互相干擾（鎖檔、同步延遲、甚至弄亂 `.git`）。之後所有 Git 操作都對這個本機路徑做，原本雲端硬碟那份留著當純備份。

```powershell
mkdir C:\Projects
# 手動複製資料夾到 C:\Projects\<專案名>，或用檔案總管拖曳複製
```

### Step 1 — 準備三個 repo 骨架檔案

在專案資料夾裡放進這三個檔案（範本見文末附錄）：

| 檔案 | 用途 |
|---|---|
| `.gitignore` | 排除大型檔案、暫存、輸出，保持 repo 精簡 |
| `README.md` | 專案說明與飛輪運作原理 |
| `CLAUDE.md` | 撰寫規範＋目前狀態，Cloud/Local session 都會自動讀取 |

`CLAUDE.md` 是關鍵——推上去之後，不管 Local 還是 Cloud session 連到這個 repo，Claude 都會自動讀到規範跟現況，不用每次重講一次。

### Step 2 — 初始化 Git 並第一次 commit

在專案資料夾開 Terminal，依序執行：

```powershell
git init
git add .
git commit -m "Initial commit"
```

> 常見錯誤：漏跑 `git add .` 直接 commit，會出現 `nothing added to commit but untracked files present`。`git add .` 是把檔案加進 staging area，一定要先做。
>
> `warning: LF will be replaced by CRLF` 只是換行符號提示，不是錯誤，忽略即可。

### Step 3 — 在 GitHub 建立空的 Private repo

登入 github.com → 右上角 **+** → **New repository**：

- Repository name：填 `<專案名>`
- 選 **Private**
- **不要**勾 "Add a README file"（避免等下 push 衝突）
- 點 **Create repository**

### Step 4 — 推上 GitHub

回到 Terminal，依序執行（把 `<你的帳號>` 換成實際帳號）：

```powershell
git remote add origin https://github.com/<你的帳號>/<專案名>.git
git branch -M main
git push -u origin main
```

第一次 push 可能會跳瀏覽器要你登入 GitHub 授權。推完看到 `[new branch] main -> main` 就成功了。

### Step 5 — 授權 Claude GitHub App 存取這個 repo

這步跟「Settings → Connectors 的 GitHub 打勾」是兩回事。Connectors 打勾只是帳號層級的基本連接；要讓 Cloud session 真正讀寫 repo，需要安裝 **Claude GitHub App** 並授權存取。

- 在開 Cloud session 選 repo 時若出現 "No repos match / Repo missing"，點裡面的 **Install the Claude GitHub app** 連結
- Account 選你的帳號
- Repository access 建議選 **Only select repositories**，只勾這一個 repo（範圍愈小愈安全，之後要加隨時能回來加）
- 點 **Install & Authorize**
- 過程中 GitHub 可能要求二次驗證（sudo mode），選 email 收驗證碼最省事

> 注意：授權權限含 **Read and write access to code / pull requests / workflows**。這是 Cloud session 能 push 回去所必需的，但也代表要謹慎控管授權範圍。

### Step 6 — 建立 Cloud environment

在開新 session 的畫面，點下方帶雲朵圖示的環境標籤 → **Add cloud environment**，填四個欄位：

| 欄位 | 填法 |
|---|---|
| **Name** | 填 `<專案名>`，方便辨識 |
| **Network access** | 維持 **Trusted**（已涵蓋 GitHub、各大套件庫、`packages.microsoft.com` 等） |
| **Environment variables** | **留空**。這裡沒有 secrets 保管機制，任何用這環境的人都看得到，不要放 API Key 或憑證 |
| **Setup script** | 依專案語言需求填（見下方） |

**Setup script 的通用寫法原則**——每個安裝步驟拆開，不要用 `&&` 串一長串：

> 血淚教訓：如果用 `wget ... && apt update && apt install ...` 串起來，只要 `apt update` 因為任何無關的 PPA 回應 403，整條 `&&` 鏈就會在 `apt update` 斷掉，後面的安裝根本不執行，卻又被 `|| echo` 吞掉、`exit 0` 讓系統誤以為成功。正確做法是每步獨立成行，`apt update` 後面單獨接 `|| true`。

**PowerShell 專案範例**（已驗證）：

```bash
#!/bin/bash
wget -q https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb -O /tmp/packages-microsoft-prod.deb
dpkg -i /tmp/packages-microsoft-prod.deb
apt update || true
apt install -y powershell || echo "PowerShell install failed, check manually"
exit 0
```

**若專案只用預裝工具（Python / Node / Go / Rust / Java / Ruby / PHP / Docker）**：Setup script 可以留空，或只放專案專屬的額外套件安裝。這些語言已內建，不用裝。

**Setup script 三大鐵則**：
- **結尾要 `exit 0`**：非零退出會直接讓 session 起不來
- **總執行時間控制在約 5 分鐘內**：超過快照建不起來
- **需要網路的安裝**：Trusted 已涵蓋主要套件庫；若設 None 網路，安裝會失敗

### Step 7 — 開 session 並驗證

點 **+ New**，環境選 `<專案名>`、repo 選 `<專案名>`、分支選 `main`。第一句先驗證環境，不要直接進正題：

```
幫我確認 <工具> 是否已經預先安裝好（例如 pwsh --version），不要自己手動安裝。
然後用 AST/語法檢查驗證主程式，再實際跑一次確認能正常執行、零錯誤。
```

**驗證通過的標準**：
- ✅ Claude 直接回報版本號，**沒有**出現「未安裝，正在安裝」→ Setup script 正確、快照生效
- ❌ 若又說「未安裝，需要先安裝」→ Setup script 有問題，回 Step 6 檢查是不是又被 `&&` 串死了

---

## 搬完之後的日常使用模式

- **Cloud session（主力）**：日常迭代都在這裡，跨裝置接續
- **Local 桌面（輔助）**：僅做臨時、單機的小幅 code 修改；**改完務必 `git add` → `git commit` → `git push` 回 GitHub**，避免版本分歧
- **驗證環境是否更新**：改過 Setup script 後，要開「全新 session」才會重跑（resume 不會），用 Step 7 的方式驗證

---

## 附錄：三個骨架檔案範本

### `.gitignore`（依專案調整副檔名）

```
# 大型媒體 / 資料檔（不適合放進 Git）
*.wav
*.mp3
*.mp4
*.zip

# 暫存與輸出
*.tmp
*.log
output/
temp/
*.bak
*.bak.*

# Claude Code 本機專屬
.claude/worktrees/

# 系統 / 編輯器雜項
.DS_Store
Thumbs.db
*.swp
```

### `CLAUDE.md`（骨架，實際內容依專案填）

```markdown
# 專案脈絡（Local / Cloud session 都會自動讀取此檔）

## 這是什麼
<一兩句話說明專案核心與主程式檔名>

## 撰寫規範（新增或修改程式碼一律套用，不因為「新寫的」就跳過）
- <把你的 coding style 逐條列出，例如註解語言、訊息嚴重度制、相對路徑、
  版本歷程更新、分隔線分節、AST 驗證、全域掃 bug、CLI+HTML 雙軌 log 等>

## 目前狀態
- <目前版本號、最近一次重大異動摘要，讓 Claude 一進來就知道進度>

## 檔案角色
- <主程式、設定檔、資料檔各自的定位與唯一真實來源>

## 開發模式提醒
- Cloud session 為主力迭代場所，跨裝置接續
- Local session 僅做臨時小幅修改，改完務必 commit + push 回 GitHub
```

### `README.md`（骨架）

```markdown
# <專案名>

> <一句話定位>

## 專案定位
<這個 repo 只放產品本身，不放對話紀錄/暫存/log。列出核心檔案與用途表格>

## 飛輪運作原理
<列出這個專案的迭代迴圈：輸入變更 → 處理 → 驗證 → 版本紀錄 → 存檔推送>

## 開發模式
- Cloud session（主力）
- Local session（輔助，改完務必 push）

## 版本
目前版本：見主程式內嵌版本變數
```
