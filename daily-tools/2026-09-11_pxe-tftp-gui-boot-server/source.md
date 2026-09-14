# PXE_TFTP_GUI_Windows.py：一支零依賴 Python 檔，在 Windows 上生出可視化 TFTP 伺服器，撐起 PXE 網路開機的最後一哩路

`PXE_TFTP_GUI_Windows.py` 是一支單檔 Python 3.10+ 工具，用 tkinter 包了一個從 socket 層自己刻的 TFTP（RRQ／octet 模式）伺服器，讓 Windows 機器不用另外裝 Tftpd64、SolarWinds TFTP 之類的第三方軟體，就能立刻起一台供 PXE 網路開機用的 TFTP 服務，並把整個開機檔傳輸過程可視化成一個可以按 Start／Stop 的小工具。

## 為什麼值得關注

PXE 網路開機（不管是灌 OS、跑 Live 診斷映像還是無人值守部署）在流程上永遠卡在同一個環節：DHCP 給了 Option 66/67 之後，還需要一台 TFTP 伺服器把 bootfile 真的送出去。在 Windows 環境下，工程師的選項通常是裝一套第三方 TFTP server、或是額外拉一台 Linux 開 `dnsmasq`／`tftpd-hpa`。這支腳本把「TFTP 伺服器」這件事直接用 Python 標準函式庫在 UDP/69 上實作出來，複製到任何裝了 Python 的 Windows 機器就能跑，不需要安裝任何套件，對需要臨時架測試環境、或不想在客戶機器上裝額外軟體的 SIT／驗機場景特別實用。

## 技術重點（實際讀過原始碼）

- **TFTP 協定是自己刻的，不是包一層現成套件**：`TFTPServer` 直接在 UDP/69 `bind()`，手動解析 RRQ 封包（opcode、filename、mode），並依 RFC 1350 用 512 bytes 為一個 block 送 DATA、等 ACK，是貨真價實從 socket 層寫出來的最小可用 TFTP（RRQ-only，不支援 WRQ 寫入），對方送 WRQ 或其他 opcode 一律回 `ERROR code=4 Illegal TFTP operation`。
- **每個請求開一條獨立資料通道**：`_serve_file()` 在收到 RRQ 後，另外開一個新的 ephemeral UDP socket 專門處理這次傳輸（符合 TFTP 標準行為：控制在 69、資料走臨時埠），並帶重傳邏輯——1 秒逾時未收到對應 block 編號的 ACK 就重送，最多重試 10 次才放棄，同時即時算出 Mbps 顯示傳輸速率。
- **路徑安全檢查沒有省略**：檔名開頭的 `/` 會被去掉、含 `..` 的路徑直接回 `Access violation`，最後還會用 `abspath.startswith(self.root)` 二次確認解析後的絕對路徑沒有跳出 TFTP-Root，避免客戶端用路徑穿越去讀伺服器上不該讀的檔案。
- **NIC 偵測用「connect-trick」而非額外套件**：`list_local_addrs()` 沒有引入 `psutil` 之類的套件抓網卡清單，而是靠對 `8.8.8.8:80` 開一個 UDP socket 再讀 `getsockname()` 拿到系統路由選出的主要對外 IP，加上 hostname 解析與固定帶入 `127.0.0.1`，是個輕量但實用的零依賴 trick。
- **CLI 提示與雙格式留痕**：GUI 內建「DHCP Helper」對話框，直接把 Option 66/67 該填什麼、UEFI 用 `bootx64.efi`／`ipxe.efi`／`grubx64.efi`、BIOS 用 `pxelinux.0` 的對應規則列出來；每次執行的 log 同時寫成純文字檔與帶內嵌 CSS 的 HTML 報告（`PXE_Overview.html`），跟同一批作者（Albert.Chou）其他工具一致的「跑完就要有留痕證據」設計語言相同。
- **刻意不做 DHCP**：程式頭部註解明講「DHCP 不在本工具內」，工具的定位很精準——只解決 TFTP 這一段，DHCP 交給既有的路由器／Windows Server／Tftpd64 處理，避免跟網路上既有 DHCP 服務衝突。

## 可延伸應用角度

- 可以跟先前收錄的 `SUSE15SPX` 部署系列工具串成一支「Windows 主機當 PXE 跳板，一鍵佈署 Linux 裸機」的完整實機示範，從 DHCP Option 設定講到這支工具起服務、再到目標機開機拉檔案的全流程。
- 拿這支工具跟 Linux 上 `dnsmasq`／`tftpd-hpa` 的 PXE 設定方式做對比，凸顯「不想動 Linux 環境時的 Windows 替代方案」這個切角。
- 從 socket 層手刻 TFTP 協定這件事本身就適合做成一支「網路協定從零實作」的教學影片，用這支工具的 `_serve_file()` 逐行對照 RFC 1350 講解 opcode、block、ACK 重傳的設計。
