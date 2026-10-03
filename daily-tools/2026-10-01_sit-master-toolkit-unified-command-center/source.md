SIT Master Toolkit（又名 SIT Unified Command Center）是一套橫跨 Windows / Ubuntu / RedHat / SUSE 四個平台、位元組對位實作的系統整合測試工具箱，把系統盤點、韌體檢查、PCIe 拓樸視覺化與 AMD Validation Toolkit（AVT）流程管理整合進同一支互動式選單腳本。

## 這是什麼

每個平台各有一支單檔腳本（Ubuntu 的 `SIT_Master_Toolkit_Ubuntu.sh`、RedHat 的 `SIT_Master_Toolkit.sh`、SUSE 版、Windows 的 `SIT_Master_Toolkit_Win2025.ps1`），透過主選單提供兩大模組：「System Inventory & Firmware」盤點 OS/CPU/DIMM/Storage/Network/GPU/PCIe/USB/BIOS/BMC 等項目並逐項判定 PASS/WARN/ERROR；「AVT Workflow Manager」管理驗證流程的 pre-test 清空日誌、post-test 收斂報告。每次執行輸出一個帶時間戳記的結構化資料夾（`reports/logs/json/debug`），同時產生 CLI 彩色輸出與可離線開啟的 dark-theme HTML 報告。

## 為什麼值得關注

這份候選原本在 9/26 因為淺層掃描只看到 `.claude`、`For_Linux`、`For_Windows` 這幾個資料夾名稱、判斷不出內容而被暫緩收錄；這次完整下載整個目錄樹後，才發現裡面其實是一個開發到 v3.5.7（Ubuntu）/ v3.6.3（Windows）、歷經十幾個版本迭代的成熟工具，光是 Ubuntu 版 README 的版本紀錄就有 11 條，每一條都附實機驗證細節。更值得注意的是，它明顯是之前已經個別收錄過的幾個主題——NVQual 的交付資料夾結構、WMI 硬體盤點、idle/C-state 監控、PCIe 相關檢查——在同一個團隊裡逐漸長成的「統一指揮中心」，是觀察工具生態系如何從零散腳本收斂成單一入口的具體案例。

## 技術重點

- **結構化輸出比照 NVQual 交付規格**：`reports/`（HTML 報告、CSV、SpreadsheetML `.xls`、`topology_report.html`）、`logs/raw/`（`lspci -vvnn`、`dmidecode`、`dmesg`、`dpkg -l` 等未整形底稿）、`json/`、`debug/`，四個平台的資料夾結構完全一致，方便跨機型統一上傳與比對。`.xls` 刻意採 SpreadsheetML 2003 XML 純文字格式而非真正的 `.xlsx`，用意是在「零依賴」前提下讓 Excel 能直接雙擊開啟，不需要 openpyxl/pandas 之類的套件。
- **D3 互動式 PCIe 拓樸圖，且刻意只標紅端點不標橋接**：階層關係由 Linux 版純 bash 走訪 `/sys/bus/pci`（Windows 版改用 `Get-PnpDevice` + `DEVPKEY_PciDevice_*` 屬性鏈，是對 `/sys` 的對應實作）建立，自動剪除 AMD Data Fabric、Dummy Bridge 等雜訊。降速判定的設計理由寫得很清楚：root port 的協商速度本來就會自然對齊下游裝置（例如 Gen5 x16 接 x4 NVMe 會協商成 x4），若把橋接也標紅會造成大量假陽性，所以只在端點的 `LnkSta < LnkCap` 時標紅——这是一個「指標設計要對齊物理行為、不能照字面套用」的範例。D3 函式庫整個內嵌進 HTML（而非讀 CDN），讓隔離網路的測試機也能直接開啟報告，代價是腳本膨脹到 350KB 以上。
- **PCI ID Resolver 子模組：五層備援、永不輸出空白或純 hex**：獨立的 `pci_id_resolver.sh` 可被主腳本 `source` 掛載，解析順序是系統 `pci.ids` → 離線快照 → 自建 OEM 對照表（`VID:DID:SVID:SDID` 比對）→ 預設關閉的線上查詢（命中會寫回快照）→ 僅廠商名稱的最後備援（並把未解析 ID 記入去重佇列檔）。這個設計精確命中一個實務痛點：伺服器廠的子系統 ID 常常沒登記在公開的 `pci.ids` 辭典裡，純粹更新辭典解決不了，必須有「自建對照表」這一層。
- **PSU 盤點的真實除錯軌跡**：v3.5.6 先用 `dmidecode -t 39` 讀 SMBIOS Power Supply 資訊，v3.5.7 的更新記錄寫明在 Tyan/MiTAC S8056 實機上驗證發現 SMBIOS Type 39 是空的、BMC FRU 也沒有 PSU 記錄，於是改抓 `ipmitool sdr` 的 PSU0/PSU1 遙測（Presence/Temp/Fan/Pin/Pout）做 fallback——這種「先寫一版、實機驗證後發現资料源不存在、再換一條路径」的版本演進，在 README 的 Changelog 裡逐條留了下來，是少見的、把除錯過程直接文件化的例子。
- **GPU 韌體盤點**：`lspci` 本身讀不到 VBIOS 版本，必須靠 `nvidia-smi`（NVIDIA）或 `rocm-smi`（AMD）才能補上韌體版本、driver 版本與序號；兩個平台的 README 都附了完整的驅動安裝教學，包含離線機要在有網路的電腦先下載 `.run` 檔、安裝前必須切到文字模式避免 GNOME 佔用 `nvidia_drm` 模組導致卡死的實戰細節。

## 可延伸應用角度

適合做成「工具如何長大」系列影片的素材：拿這套 Unified Command Center 跟之前個別介紹過的 NVQual（USB 黑名單）、WMI 硬體盤點、C-state 監控分開對照，講一個真實測試團隊的腳本是怎麼從點狀工具逐漸收斂成單一入口的指揮中心。也適合做跨平台實作對照的技術影片——同一套 PCIe 拓樸降速判定邏輯，Linux 版走 `/sys/bus/pci`、Windows 版走 PnP `DEVPKEY`，兩邊功能對齊但資料來源完全不同，是很好的「跨平台抽象層設計」教材。PCI ID Resolver 的五層備援設計也可以單獨拉出來做一支「容錯設計模式」短片。
