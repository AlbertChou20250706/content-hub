NTP 客戶端／伺服器佈署工具包是一套鎖定 SIT 隔離測試網路的時間同步腳本組合，涵蓋 Linux（RHEL/SUSE/Ubuntu）與 Windows 被測端的一鍵同步，以及 Windows NTP Server 端「機碼遺失」時的登錄檔搶救流程。

## 為什麼值得關注

SIT 測試環境經常是對外隔離的網路，機器連不到公網 NTP pool，只能仰賴內部自架的 Windows NTP Server（192.168.10.10）當時間基準。時間沒對齊在測試現場是很麻煩的坑：Log 時間軸兜不起來、憑證/簽章驗證因時間偏差失敗、多台機器的事件關聯全部對不上。這套工具把「佈署新環境時要重新設定時間同步」這件事，從手動查指令、常常忘記步驟，變成一行指令帶 IP 參數就能跑完，而且連 Windows Server 端本身環境壞掉（`NtpServer` 機碼遺失）都準備好了搶救流程。

## 技術重點

實際下載並讀過資料夾內容後，重點如下：

- **雙版本 Linux 腳本並存**：根目錄的 `setup_ntp_linux.sh` 是較早的多發行版通用版（RHEL/SUSE/Debian 共用一支，用 `yum install || zypper install || apt-get install` 依序嘗試），而 `ubuntu/setup_ntp_ubuntu.sh` 是 2026-05-12 才寫的 v2.0.0 Ubuntu 專版：多了先停用 `systemd-timesyncd`（Ubuntu 預設的時間同步服務，會跟 chrony 打架）、結構化的帶顏色 log 函式（INFO/PASS/ERROR/FAIL 四級，同時寫到檔案與螢幕）、以及同步後額外執行 `hwclock -w` 把系統時間寫回硬體時鐘。是典型的「先求有、再依實際環境優化」的腳本演進軌跡。
- **Windows 端的「大時間差強制同步」設計**：`setup_ntp_windows.bat` 明確把登錄值 `MaxPosPhaseCorrection`／`MaxNegPhaseCorrection` 改成 `0xFFFFFFFF`（無上限）。這是因為 Windows 內建的 w32time 服務預設只接受落在一定範圍內的時間校正，超過閾值會直接拒絕同步；SIT 環境裡機器常常斷電很久或剛裝好系統，時間差可能是好幾天，不先解除這個限制，光靠 `w32tm /resync` 根本校正不過去。
- **NTP Server 端的登錄檔搶救 SOP**：README 記錄了一個具體故障案例——Windows NTP Server 遺失 `TimeProviders\NtpServer` 機碼後要怎麼從零重建（`Enabled`/`InputProvider`/`DllName` 三個值、把 `AnnounceFlags` 設為 5 使其成為權威來源、`Type` 設為 `NTP`），並額外處理了一個容易被忽略的硬體層雷區：虛擬機環境下 Intel 網卡若沒關掉 UDP/IPv4 Checksum Offload，會讓 NTP 封包在網卡層被誤判校驗和錯誤而丟包。`Server_Fix.ps1` 把整套修復自動化成一支腳本，用 `Disable-NetAdapterChecksumOffload` 統一關閉。
- **驗證與留痕**：Linux 端固定寫 log 到 `/root/Documents/log/ntp_*.log`，Windows 客戶端則寫到 `C:\Users\Public\Documents\SIT_Logs`，方便測試工程師事後稽核「這台機器到底有沒有真的做過時間同步」。

## 可延伸應用角度

- 適合搭配既有的「Windows/Linux 環境佈署自動化」系列（例如 Configure_kdump_halt、SPD_Flash 這類 SIT 現場工具），做成「新機上架 SOP 自動化」的其中一集，強調時間同步這種容易被忽略、但會拖累整條測試流程的環節。
- 可以延伸講「Windows w32time 的時間校正限制」這個冷知識，搭配 MaxPhaseCorrection 的真實踩坑案例，適合做成排錯（troubleshooting）類短影音。
- 兩支 Linux 腳本（多發行版通用版 vs Ubuntu 專版）的差異，也適合拿來當「同一支腳本如何隨著實際使用環境反覆優化」的迭代案例。
