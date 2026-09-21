`SPD_Flash`（Linux 分支核心腳本 `enter_manuf.sh` + `flash_spd_manual.sh`，前身為單一支 `spd_flash.sh` v1.0.0 → v1.1.1）是一套把 Fujitsu Primergy 伺服器「寫入 DIMM SPD（Serial Presence Detect）資料」這件事腳本化的工具：先切換系統板進入製造模式並自動重開機，重開機後再用原廠工具 `SMBVIEW64` 把 `memory.ini` 裡定義的 SPD 內容實際寫進記憶體模組，全程輸出 TXT + HTML 雙份帶時間戳的日誌。

## 為什麼值得關注

寫入 SPD 跟「讀取／驗證 SPD」是完全不同等級的風險操作——這份素材裡的 `memory.ini` 只有一行 `MemMark=1`，但原廠規則是「一旦寫入這個標記，同一次開機內就讀不到 SPD 了，必須先重開機才能再讀」。也就是說，操作本身會讓自己短暫失效，而且失效期橫跨一次 reboot，跟同一批素材裡 `SMBVIEW_verify_Memory_for_SCD`（純讀取／收集 SPD 的 Python 工具）面對的是同一套原廠限制，但這裡處理的是「寫」而不是「讀」，風險與操作順序要求更高：如果在 SPD 還沒被正確寫入、或者忘記讓系統重開機的狀態下就去驗證，很容易誤判成失敗。這支工具把「切製造模式 → 重開機 → 部署 INI → 嘗試多種 SMBVIEW64 語法寫入 → 視需要驗證」這一串手動步驟，拆成兩支各司其職、可以分別重跑的腳本。

從資料夾裡的版本演進也能看到作者對這套流程理解的變化：`Previous/SPD_Flash_v1.0` 是一支單腳本一次做完全部流程（含 systemd 服務在重開機後自動續跑），`Previous/SPD_Flash_v1.1.1` 修正了 `SMBVIEW64` 的呼叫語法並加強了回傳碼檢查；但現行主線 `Linux/SPD_Flash_v1.0` 反而放棄了「單腳本自動續跑」的設計，改回兩支各自手動執行的腳本（`enter_manuf.sh` 與 `flash_spd_manual.sh`）。這是一個「先追求全自動，後來為了可控性和除錯方便退回半自動」的真實工程取捨案例。

## 技術重點（實際讀過原始碼）

- **`flash_spd_manual.sh` 對 `SMBVIEW64` 的命令列參數用「窮舉多種語法直到成功」而非硬寫死一種**：腳本內建 `-INI="SMBVIEW.INI"`、`-INI=SMBVIEW.INI`、`-INI="memory.ini"`、`-INI memory.ini`、`-ini=SMBVIEW.INI`，最後還有一個空字串（no-arg，讓 `SMBVIEW64` 自己去讀預設的 `SMBVIEW.INI`）作為 fallback，逐一嘗試直到某個語法回傳 `rc=0` 就停止（第 122–147 行）。README 裡也直接點名某些版本的 `SMBVIEW64` 會忽略小寫 `-ini`，必須用大寫 `-INI`——這是從實際除錯經驗回寫進程式邏輯的典型案例。
- **INI 部署前會先做 CRLF 轉換與雙檔名部署**：`install -m 0644 -D` 把使用者的 `memory.ini`複製進 `SMBVIEW64` 所在目錄，同時存成 `memory.ini` 與 `SMBVIEW.INI` 兩個檔名（因為不同版本的 `SMBVIEW64` 讀取的預設檔名不一致），並用 `sed -i 's/\r$//'` 去除 Windows 換行符，避免原廠工具因為 CRLF 而報 `Read INI file failed`。
- **`enter_manuf.sh` 與 `flash_spd_manual.sh` 明確拆成「必須先做」與「重開機後才能做」兩個階段**：`enter_manuf.sh` 只負責 `sobcontrol start` → `Pc_Ident64 flags.w=manuf` → 立即 `reboot`（優先呼叫 `/sbin/reboot`，找不到才退回 `reboot`）；`flash_spd_manual.sh` 完全不管重開機這件事，只信任使用者已經在正確時機執行它，兩支腳本之間沒有自動狀態檔銜接——這跟同批素材裡 `SMBVIEW` 收集工具用 `/proc/stat` 的 `btime` 自動判斷是否已重開機的設計形成有趣對照：一個選擇用程式自動驗證狀態，一個選擇把「時機正確」的責任交還給操作者換取腳本簡單、好除錯。
- **兩支腳本都內建 `--dry-run`，且 dry-run 模式下連 GABI 裝置節點檢查都會跳過真正的硬體動作**：只印出「將會執行什麼指令」而不實際呼叫 `sobcontrol`／`Pc_Ident64`／`SMBVIEW64`，方便在不具備硬體或不想真的觸發重開機的情況下先確認參數與路徑是否正確。
- **GABI 裝置節點（`/dev/gabi*`／`/dev/sob*`）缺失時只警告不中止**，README 的疑難排解章節記錄了一個已知環境限制：SUSE 上原廠 GABI 驅動常常不支援，`SMBVIEW64` 會卡在印出版本號之後就不動，遇到這種情況官方建議的解法是換一台原廠支援的發行版（RHEL/Rocky/Alma）LiveUSB 來執行，而不是在 SUSE 上debug 驅動相容性。
- **日誌固定寫死輸出到 `/root/Documents`**，每次執行都會產生帶時間戳的 `.log`（純文字）與 `.html`（把 log 內容跑過 HTML escape 後包進 `<pre>`）雙份紀錄，方便直接把 HTML 附進測試紀錄或工單。

## 可延伸應用角度

- 可以和同批素材的 `SMBVIEW Memory Collector`（Python，讀取／驗證 SPD）放在同一個系列裡對照：同樣的原廠二進位工具（`SMBVIEW64`、`Pc_Ident64`、`sobcontrol`）、同樣的「製造模式寫入後同開機內讀不到」限制，一邊選擇用 `btime` 自動判斷重開機狀態走向全自動，一邊選擇拆成兩支手動腳本走向半自動——很適合做成「同一個硬體限制，兩種完全不同的自動化取捨」比較影片。
- `SMBVIEW64` 命令列語法窮舉、CRLF 清理、雙檔名部署這幾個技術點，適合剪成「包裝原廠二進位工具時最常踩的相容性地雷」短片，跟 `SUSE15SPX` SUT Toolkit、SMBVIEW Memory Collector 屬於同一主題，可以串成一個小系列。
- 「先做全自動（systemd 續跑），後來退回半自動（兩支手動腳本）」這個版本演進本身就是一個很好的工程決策案例，適合搭配「什麼時候該追求全自動、什麼時候該保留人工確認點」的維運腳本設計討論。
