`Configure_kdump_halt`（腳本本體 `configure_kdump_halt_v1.1.sh`）是一支跨 RHEL／SUSE 發行版的 kdump「當機後行為」設定工具：把 Linux 系統遇到 kernel panic 時預設會自動 reboot、導致崩潰現場（vmcore、`/var/log/messages` 裡的 panic 訊息）被覆蓋或遺失的問題，改成透過 `final_action halt`（RHEL）或 `KDUMP_IMMEDIATE_REBOOT="false"`（SUSE）強制系統停在崩潰當下，讓工程師有機會採證。

## 為什麼值得關注

這是一個從真實內部缺陷單直接長出腳本的案例。資料夾裡附的 `Bug.txt` 記錄了原始踩雷過程：「目前我們內部發現在 linux OS（OS 預設）下若發生 kernel panic 的時候，系統並沒有停住，而是會 reboot」，因此要求測試人員「跑完 cycle 的時候，務必要檢查 `/var/log/message` 和 test log 須附上此檔案」，而且「三個案子都要」手動重複同一個操作：「請你們裝完系統之後，需要 Linux OS 手動編輯 `/etc/kdump.conf`，加入 `final_action_halt`」。這支腳本就是把那條寫在缺陷單裡、每個案子都要人工重做一次的 SOP，變成一個可重複執行、跨平台的自動化工具——RHEL 和 SUSE 的設定檔路徑與參數完全不同，人工操作很容易漏掉平台差異，腳本內建了分支判斷就不會出這種錯。

## 技術重點

實際下載並讀過 `configure_kdump_halt_v1.1.sh`、`ReaMe.txt`、`Bug.txt` 後，重點如下：

- **跨發行版分支**：用 `/etc/os-release` 的 `ID`／`ID_LIKE` 判斷是 RHEL 系還是 SUSE 系，走完全不同的設定檔與 key——RHEL 改 `/etc/kdump.conf` 的 `final_action`（可選 `failure_action`）；SUSE 改 `/etc/sysconfig/kdump` 的 `KDUMP_IMMEDIATE_REBOOT="false"`（可選附加 `KDUMP_POSTSCRIPT="/usr/bin/systemctl halt -f"` 當二次保險）。
- **`ensure_kv()` 冪等寫入**：用 `grep -Eq` 判斷 key 是否已存在於設定檔，存在就用 `sed -i -E` 原地取代，不存在則直接 append，可重複執行不會把設定寫成兩份重複的 key。
- **改設定前一定先備份**：`backup_file()` 用 `cp -p` 複製成帶時間戳記的 `xxx.bak.<timestamp>`，改壞了可以直接還原，這在會動系統開機／崩潰行為的腳本上是必要的安全網。
- **旗標設計**：`--dry-run`（只印出將要做的變更，不動手）、`--include-failure`（RHEL 額外設定 `failure_action halt`）、`--set-sysctl`（額外寫入 `kernel.panic = 0` 到 `/etc/sysctl.d/99-kernel-panic.conf` 並立即 `sysctl -p` 套用）、`--suse-post-halt`（SUSE 加上 POSTSCRIPT 呼叫 `systemctl halt -f`）。四個旗標對應四種常見的「還要更嚴格一點」需求，不用改程式碼就能組合。
- **收尾動作**：設定寫完後自動判斷用 `kdumpctl restart` 或 `systemctl restart kdump.service` 重建 initramfs，再呼叫對應指令印出目前 kdump 狀態，一次跑完「改設定 → 生效 → 驗證」全流程。
- **這支腳本後來被整合進更大的工具**：`ReaMe.txt` 的版本紀錄顯示，這個「加入 `final_action halt`」的邏輯後來被併進一支叫 `RebuildEvents.sh` 的更大自動化腳本（v2.3 → v2.4r1），額外加上非 OS 磁碟自動格式化掛載、log 清除、用 `ipmitool sel clear` 清 BMC SEL 紀錄等功能，而且演進過程明顯看得出安全意識在提升——v2.4 開始預設 `dry-run`、要加 `--confirm` 才會真的動手格式化磁碟，並且用 `lsblk` + `df` 自動排除系統根磁碟 `/dev/sda`，避免腳本誤刪系統碟。

## 可延伸應用角度

- 適合放進「RAS／崩潰驗證系列」，跟先前已收錄的 `kdump-helper.sh`（負責確保 kdump 機制本身有開、crashkernel 記憶體有保留、能不能真的觸發並產生 vmcore）放在一起做對照：`kdump-helper.sh` 管的是「dump 機制打不打得開」，這支 `Configure_kdump_halt` 管的是「dump 產生之後系統該停在那邊還是自動重開機」，兩支合起來剛好是一條完整的當機採證鏈，很適合做成上下集對照影片。
- `Bug.txt` 這份「缺陷單原文變成腳本邏輯」的素材非常適合做一集「怎麼把 QA 手動 SOP 自動化成腳本」的個案拆解，直接對照缺陷單裡的中文操作要求和程式碼裡對應的判斷式。
- `ReaMe.txt` 揭露的版本演進（v1.0 單純加設定 → v1.1 加 SUSE 支援 → 整合進 `RebuildEvents.sh` v2.3 → v2.4r1 加上 dry-run／`--confirm`／排除根碟等安全防呆機制）可以延伸做一集「工具怎麼從單一功能小腳本，長成有安全防呆機制的自動化工具箱」，用同一份工具的四個版本當時間軸範例。
