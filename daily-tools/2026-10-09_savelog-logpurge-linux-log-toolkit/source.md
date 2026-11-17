# SystemEvent（Linux）：SaveLogTool 日誌收集與 LogPurge_Tool 清除工具組

> 來源路徑：`SystemEvent/Linux/`（`SaveLogTool_v1.9.0.sh`、`LogPurge_Tool.sh` v2.5.0）
> 設計者：Albert Chou

## 這是什麼

兩支配套但用途完全相反的 Bash 工具：**SaveLogTool** 收集 `/var/log` 日誌與一份完整的硬體快照、產生 CSV／HTML／雜湊 manifest；**LogPurge_Tool** 則是清除特定日誌路徑並清掉 BMC 的 System Event Log（SEL），讓測試機的事件紀錄從 WebUI 上消失。兩者刻意分開維護，用意是「先留證據、再清場地」，不是同一個操作的兩個開關。

## 為什麼值得關注

這個資料夾裡藏了一個很具體、已經實際修好的部署事故：`install_savelog_service.sh` 原本裝出來的每日排程（systemd timer）**保證會失敗**，而且是兩個獨立問題疊加——`ExecStart` 帶了主程式根本不支援的 `--no-zip` 參數，就算拿掉這個參數，排程腳本也只把 wrapper (`savelog.sh`) 複製到 `/usr/local/bin`，沒把真正的主程式 `SaveLogTool_v1.9.0.sh` 一起裝過去，而 wrapper 的邏輯是「在自己旁邊找同資料夾版本最高的主程式」，所以排程一執行就是 `No SaveLogTool_v*.sh found beside this wrapper.`。這代表這組「每日自動收集日誌」的保險機制，從裝上去那天開始就從沒真正跑過——這種「部署腳本本身沒被驗證過」的坑在 SIT 環境很常見，2026-09-24 的修正記錄把整個排查與修法過程都寫進了 README，是個很好的「安裝腳本本身也要測試」的真實案例。

另一個值得注意的地方是同一個資料夾裡**同時存在兩組都自稱 v1.9.0、日期相同，但 SHA256 不同的檔案**（頂層散放的腳本 vs. `SaveLogTool_v1.9.0/` 資料夾內的腳本）：頂層版本多了 `--include-glob`／`--exclude-glob`／`-q`／`-V` 等參數，排程改成開機 5 分鐘後、每 12 小時跑一次；資料夾版本則是每天固定 03:30、且已經修好上述的安裝 bug。README 自己老實列出兩者差異與尚待整理的版本號衝突，而不是假裝只有一份——這對「如何老實記錄技術債」是個不錯的示範。

## 技術重點

實際下載並讀過整個資料夾（含歷史版本 `Provious/`）後，幾個值得注意的設計：

- **零參數即可執行**：不帶任何旗標直接跑，預設來源 `/var/log`、輸出 `/root/Documents/Logs`；來源不可讀時會自動退回 `/etc`，避免因權限問題整個失敗。
- **收集與清場完全分離**：SaveLogTool 执行時把來源複製到 `/tmp` 暫存，產生清單與雜湊後才刪除暫存——預設**不保留日誌原始副本**，只留 CSV／HTML／manifest，要保留原始檔要額外加 `--zip`。這個行為容易被誤會成「忘記備份」，但其實是刻意設計，README 也特別點出來提醒。
- **Legacy 硬體快照是附加價值**：除了日誌本身，`<基底>_legacy/` 還會依類別（CPU、PCIe、磁碟 SMART、dmesg、dmidecode、網路…）另外收集一份當下的硬體狀態快照，並產生可搜尋、點檔名即時 iframe 預覽的 `index.html`——相當於把「日誌收集」升級成「問題發生當下的完整現場快照」。
- **禁字掃描會誤報但不中止**：啟動時遞迴掃描目前工作目錄的 `*.sh`／`*.md` 找預設禁字，若在工具自己的資料夾內執行（`CHEATSHEET_v1.9.0.md` 本身就含有該字）一定會顯示 `CRIT`，但腳本邏輯是「顯示但不中止」，這個「看起來像失敗但其實沒事」的訊息設計，若不先讀 README 很容易被誤判成 bug。
- **LogPurge_Tool 的清除是真的不可逆、無確認**：`rm -rf` 刪除前會先用 `chattr -i` 解除不可變屬性，再清 dmesg 緩衝區，最後呼叫 `/home/ipmiview/IPMIVIEW64 clearsel` 清 BMC SEL，並用 `ipmitool sel list` 迴圈（最長 45 秒）驗證清空結果、失敗才改用 `ipmitool sel clear` 兜底。全程沒有任何互動確認——README 用整段警告強調「這跟收集無關，要保留證據請先跑 SaveLogTool」，等於把風險控制完全交給使用順序而非程式邏輯。

## 可延伸應用角度

- 很適合做成「一個排程 bug 從發現到修好」的案例影片：對照「原本設計→實測失敗訊息→根因拆解（兩個獨立問題）→修法→模擬驗證」的完整記錄，示範怎麼寫一份誠實、可追溯的修復報告，而不是只改完程式碼就結束。
- 可以跟先前收錄的《WithMaxHDDFunctionTest》系列（裡面的 `Verify_SaveLogTool_Report_v1.0.sh` 專門驗證 SaveLogTool 的輸出完整性）放在同一集比較：一支工具產生證據、另一支工具反過來驗證證據本身沒有漏，形成一套完整的「生成—驗證」鏈。
- LogPurge_Tool 的「無確認、不可逆」設計適合拿來討論自動化工具的風險分級：對照同資料夾 SaveLogTool 的保守預設（預設不留原始檔副本、遇禁字只警告不中止），兩支工具在「危險操作該不該加防護」這個設計哲學上恰好形成對比，可以當作「破壞性操作到底要不要防呆」的具體教材。
