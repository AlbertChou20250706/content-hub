# Health Guardian 雙平台健檢機器人 — 同一套「工程 DNA」怎麼在 Bash 和 PowerShell 上長成兩支不同的工具

> 來源路徑：`GitHub/20260522-20260521T234827Z-3-001/20260522/`（`Linux/v1.0.0/linux_health_guardian.sh` v1.0.0、`windows/v1.1.0/SystemHealthGuardian.ps1` v1.1.0，均建置於 2026-05-22）
> 設計者：Albert Chou　最後更新：2026-05-22

---

## 這是什麼

兩支各自獨立、但共用同一套設計規範的系統健檢工具：一支是純 Bash 的 `linux_health_guardian.sh`（跨 RHEL / SUSE / Ubuntu 三大發行版），另一支是 `SystemHealthGuardian.ps1`（帶 WPF GUI 的 Windows 健檢+告警工具）。兩者都各自輸出 CLI 文字日誌與工業風深色 HTML 報表，配色語意完全對齊（PASS 綠、WARN 黃、FAIL 紅、CRIT 洋紅）。同資料夾下附的 `README.md`（代號 Forge）把這套跨平台一致性稱為「Albert Style 五大鐵律」——這不是產品說明，而是一份工程規範文件，描述這兩支腳本是照什麼標準寫出來的。

## 為什麼值得關注

多數「順手寫的健檢腳本」只解決眼前那台機器的問題，換一個平台就得整套重寫、邏輯還會悄悄跑掉。這兩支工具刻意反著做：先定義一套與語言/平台無關的規範（相對路徑、雙格式日誌、色彩語意、版本比對、防禦性寫法），再分別用 Bash 和 PowerShell 各自實作一次，藉此驗證規範本身是否真的可跨平台複製。這對任何要維護「一套邏輯、多個環境」腳本的人（IT 維運、SIT 硬體驗證團隊）是個具體可抄的範本：規範怎麼定義、怎麼在兩種語法完全不同的語言裡落地成同樣的行為。

## 技術重點（實際看過程式碼）

- **Linux 版的健檢完全不依賴外部工具**：CPU 用 `/proc/stat` 前後兩次取樣（`sleep 0.5` 間隔）算差值使用率；記憶體用 `/proc/meminfo` 的 `MemAvailable` 而非簡單的 `Free`，避免把 buffer/cache 誤算成已用；磁碟用 `df -P -x tmpfs -x devtmpfs -x overlay -x squashfs` 主動排除虛擬檔案系統，避免虛擬掛載點污染警報。這三項全部只讀 `/proc` 或標準工具，不裝任何額外套件。
- **發行版差異被收斂到 `detect_os` 單一函式**：讀 `/etc/os-release` 判斷 `OS_FAMILY`（rhel/suse/debian），再依此解析服務名稱差異——SSH 服務在 Debian 系叫 `ssh`、RHEL/SUSE 系叫 `sshd`；cron 服務 RHEL 系叫 `crond`、其餘叫 `cron`。這個對應表若寫錯，`check_services` 會對著不存在的服務名回報假故障，是這類跨發行版工具最容易踩的坑，這裡用一個集中函式吸收掉，其他檢查邏輯完全不用感知平台差異。
- **網路探測做了三級降級**：優先用 `ping`；沒有 `ping` 就用 bash 內建的 `/dev/tcp/<host>/<port>` 開一條 TCP 連線測試（`exec 3<>"/dev/tcp/..."`，純 bash 語法、不需外部指令）；再退到 `curl`。這保證在被裁剪過的最小化容器或精簡版 Linux（可能連 `ping` 都沒裝）上也能跑通，而不是直接判定「網路異常」。
- **更新檢查有明確的逾時保護**：`dnf/yum/zypper/apt` 四種套件管理器分別呼叫，但每個呼叫都包一層 `timeout 30`，避免在 repo 連線異常、DNS 解析卡住的機器上把整支健檢腳本卡死——這是很容易被忽略、卻是巡檢腳本要能無人值守運作的必要條件。
- **PowerShell 版把同一套語意搬進 GUI**：`SystemHealthGuardian.ps1`（1050 行，26 個函式）除了 CLI/HTML 雙日誌，額外做了 `DispatcherTimer` 驅動的自動掃描迴圈與 WPF 深色介面，並在 `Send-Alert` 裡接了 Teams Webhook 與 Email SMTP——告警密鑰刻意不寫死在腳本內，改讀環境變數或外部 `alert.config.json`（本體只提供 `.example` 範本），對照 Bash 版完全不依賴外部服務的極簡路線，剛好是同一套規範在「單機巡檢」與「企業告警」兩種使用情境下的兩種取捨。
- **兩邊都用 `\033`/`[char]27` 手動預定義 ANSI 色碼，而非 `echo -e` 或 `Write-Host` 的預設色彩**：這個選擇是為了讓輸出在舊版 SSH 終端機或 Serial Console 上仍能正確顯示顏色，同時保證 CLI 與 HTML 報表用的是同一組色碼（例如 `#F59E0B` 黃色同時用在 WARN 的終端輸出與 HTML 徽章上），不會出現「終端機看到黃色、報表看到別的顏色」的不一致。

## 可延伸應用角度

- **「先定義規範、再雙平台實作驗證」這個開發流程本身**就是很好的一集主題：多數教學只講單一平台的腳本寫法，這裡有個具體案例可以拆解——同一份需求規範，Bash 和 PowerShell 各自遇到什麼取捨（Bash 走純 `/proc` 零依賴、PowerShell 走 GUI+企業告警），適合做成「如何設計可跨平台複製的維運工具規範」系列。
- **`detect_os` 集中吸收發行版差異**的寫法可以獨立做一支「Bash 寫跨發行版腳本的坑與解法」短片，搭配 `/dev/tcp` 降級探測、`timeout` 逾時保護，都是很具體、可直接照抄進其他腳本的技巧點。
- 這支工具與上週介紹過的 `SIT Storage All-Stress Toolkit`（`daily-tools/2026-08-19_sit-storage-stress-toolkit`）同樣出自 Albert Chou、同樣走「單一腳本零依賴 + CLI/HTML 雙日誌 + 色彩語意對齊」路線，可以串成同一個「Albert Style 工具設計哲學」系列，比較不同工具在同一套原則下的具體實作差異。
- 告警串接（Teams Webhook / Email SMTP）與機密零硬編碼的設計（`alert.config.json.example` + 環境變數），適合單獨做一則「腳本裡的密鑰到底該放哪」的資安衛生短片，對照常見的「密鑰寫死在腳本裡」反例。
