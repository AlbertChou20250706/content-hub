這是一份針對 Fujitsu iRMC S6 BMC 的 NTP 時間同步設定與驗證 SOP，目標是解決一個很具體、很惱人的現象：BMC 在每次 AC/DC 冷開機或重開機後，SEL（System Event Log）裡會先跳出一筆 `2E0011 – Synchronization failed`，隔一下才補一筆 `2E0012 – ok`，造成測試現場的事件紀錄多出一堆雜訊。

## 為什麼值得關注

SEL 裡的 `failed → ok` 雜訊看起來不影響功能（最後畢竟同步成功了），但在 SIT／驗證現場是個真實的麻煩：每次重開機都要人工確認這筆 `failed` 是「無害的過渡現象」還是「真的同步不上」，久而久之變成一個反覆被重新調查、卻其實已知答案的雷區。這份 SOP 把判斷標準寫死——Primary NTP Server 要填內網 IP（`192.168.101.95`）而不是 hostname（BMC 大多沒有 DNS 或連不到外網解析），這一個小改動就能讓開機後 1–2 分鐘內只出現一筆乾淨的 `2E0012 ok`，不再有 `failed` 紀錄。

## 技術重點

實際讀過下載下來的兩份文件（`README.md` 總覽版 + `iRMC_S6_NTP_SOP_v1.0_Albert.md` 一頁式操作版）後，重點如下：

- **Primary 填 IP 不填 hostname 是整份 SOP 的核心修復點**：對照「常見問題排查」表格，`failed` 後補 `ok` 的最常見原因正是 Primary 用了 hostname 但 BMC 端無 DNS／連不到外網解析，一旦改填 `192.168.101.95` 就解決。
- **Lab 側（NTP Server）準備分 Linux／Windows 兩套**：Linux 端建議用 `chrony`，`/etc/chrony.conf` 要加 `allow 192.168.101.0/24`，若實驗室網路完全離線則保留 `local stratum 10` 讓它自己當權威時間源；Windows Server 端則是改 `W32Time` 登錄檔啟用 `NtpServer` 並 `/reliable:yes`，兩邊都要記得在防火牆開 `UDP/123` 的進出站。
- **驗收標準寫得很具體，不是空泛的「同步成功」**：分成即時測試（Apply → NTP Update 後 SEL 只能出現 `2E0012 ok`）與冷開機/重開機測試（1–2 分鐘內出現新的 `2E0012 ok`，過程中不能有 `failed`），另外提供 `ipmitool bmc gettime` / `ipmitool sel list` 作為選用的 CLI 輔助檢查，方便在沒有 WebUI 的情境下腳本化驗證。
- **常見問題排查表把四種失敗模式對應到具體處置**：hostname 解析不到、NTP 服務未監聽或網段未放行、NTP 伺服器開機比 BMC 晚、時間偏差過大需先手動校時——每一條都對應一個明確動作，不是「請檢查網路設定」這種空話。

## 可延伸應用角度

- 可以跟同系列的 BMC/iRMC 設定類主題（例如 BMC_Password_Change、BMC_Fan_SKU_Tool）放在一起，做成「iRMC/BMC 現場維運 SOP 合集」的其中一集。
- `failed → ok` 的 SEL 雜訊排查很適合單獨做成一支「如何讀懂 SEL 時間同步錯誤碼」的短影音，用真實的 2E0011/2E0012 案例帶出「看起來是錯誤但其實是正常過渡」這個排錯心法。
- 也適合跟先前介紹過的通用 NTP 客戶端/伺服器工具包（SIT 隔離網路版）做對照，呈現「同樣是 NTP，BMC 層級的時間同步」跟「作業系統層級的時間同步」各自的坑點差異。
