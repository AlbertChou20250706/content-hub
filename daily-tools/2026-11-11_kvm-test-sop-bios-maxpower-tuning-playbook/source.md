## 這是什麼

這份素材來自 Drive 上的「SOP」資料夾，收錄兩份文件：一份是 Oracle Linux（UEK Kernel）環境下的 KVM 虛擬化測試 SOP，一份是 BIOS → Linux MaxPower 最佳化設定對照表。兩份都是操作步驟與設定對照文件，不含程式碼，不是可執行工具——但對做 SIT／伺服器驗證的工程師來說，是把「怎麼測、怎麼調」寫成可複製步驟的標準作業流程，價值在於流程本身而不是腳本。

## 為什麼值得關注

伺服器送測前要確認兩件事：一是這台機器的虛擬化能力（CPU/記憶體/Kernel/KVM 模組）是否真的可用，二是 BIOS 的電源管理設定是否被壓測或效能基準測試需要的「全速模式」蓋掉。這兩件事常常分屬不同工程師、不同階段處理，缺了任何一步都會讓後面的壓測數字失真或 VM 建置失敗卻找不到原因。這份 SOP 把兩者都寫成「對照表 + 檢查點」的形式，新人或換平台時可以直接照表操作，不用每次重新摸索 BIOS 選項名稱或 KVM 健檢指令。

## 技術重點

**KVM_Test_SOP.docx（General KVM Test）**
- 先決條件：安裝 `qemu-kvm`／`libvirt`／`virt-install` 等虛擬化套件、啟動 `libvirtd`。
- 分層驗證：CPU（BIOS 開 VT-x/AMD-V，`lscpu` 確認）→ 記憶體（≥4GB 可分配）→ Kernel（`uname -r` 含 uek，`kernel-uek` 已裝且為預設 kernel）→ KVM 基礎健檢（`libvirtd` 狀態、`virsh capabilities`、default storage pool 為 active）。
- 四種 VM 安裝來源都要測：CD/DVD、ISO、FTP、既有映像檔（`virt-install` 範例指令），避免只測過一種路徑就宣稱「KVM 可用」。
- 驗證點具體到指令層級：`virsh list --all`／`virsh dominfo`、VM 內部網路測試、安裝/登入畫面截圖；失敗時回頭查 `journalctl -u libvirtd` 與 qemu log。
- 測試前後蒐證流程化：用（資料夾外的）`Clear_event_log_v1.3.sh` 清場、`Save_event_log_v1.3.sh` 收集，產生 `event_log_YYYYMMDD_HHMMSS/` 目錄，讓每次測試都有可追溯的留存紀錄。

**Linux_MaxPower_BIOS_Settings.docx**
- 14 項 BIOS 選項對照表，每項附建議值與備註，核心邏輯是「關掉所有會動態降頻/降核心數的機制」：Logical Processor／Active Processor Cores 全開、保留 EIST 與 Turbo、P-State 統一為 SW_ALL、Energy Performance 設 Performance、關閉 C1E/C6 並把 Package C State limit 鎖在 C0、CPU Performance Boost 設 Aggressive、Uncore（Compute/IO）頻率鎖 Max Performance、Utilization Profile 設 Even。
- 驗證不是調完就結束：用 `stress-ng` 壓測 6 小時、`mpstat 1` 看 CPU 負載、`ipmitool sensor` 透過 BMC 監控功耗與溫度，確保設定真的生效且沒有過熱降頻。

## 可延伸應用角度

可以跟既有的 SIT 測試框架／BIOS 調校系列搭在一起，拆成兩支短影片：一支講「KVM 能力驗證該測哪四種路徑、每一步卡住要查哪個 log」，另一支講「BIOS 這 14 個選項為什麼要這樣設、怎麼用 stress-ng + ipmitool 驗證設定真的生效」。這類內容面向的是「怎麼做可重複、可交接的驗證流程」，適合搭配頻道裡偏 SOP／流程標準化的系列，而不是單純的工具評測。
