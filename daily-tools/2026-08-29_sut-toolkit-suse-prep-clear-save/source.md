# SUT Toolkit（Albert Style）— SUSE 測試機的 prep / clear / save 三合一準備工具

`sut_toolkit_albert_v3_0.py` 是一支單檔 Python CLI，專門處理「Server Under Test（SUT）」上機前的準備、清場、留證三件事：把一台剛裝好 SUSE 15 SPx 的裸機，透過一條指令設成可被實驗室排程系統（XinConf）辨識、可 GUI 遠端操作、日誌乾淨的待測狀態。

## 為什麼值得關注

SUSE 測試機重灌後要手動做的事情很瑣碎且容易漏：`hostnamectl set-hostname`、改 `wicked` 的 `ifcfg-<iface>` 讓 DHCP 回報主機名、清掉舊的 `/etc/hosts` 記錄再寫新的、重啟網路、確認正反解都通、判斷有沒有裝 GNOME 再决定要不要切 `graphical.target` 並啟用 GDM、最後打一次內部的 XinConf 狀態 API 確認這台機器真的被系統看到了。這套流程如果每次都手動操作，順序錯一步（例如网路服务重启在 hosts 写入之前）就要重跑，而且沒有留下紀錄可供交接。這支工具把整個流程收斂成 `prep` 一個子命令，並提供 `clear`（清 dmesg、清測試殘留 log、檢查 `/var/crash`）、`save`（把 `uname`／`lsblk`／`lspci`／`dmidecode`／`dmesg` 過濾結果／`os-release` 等分門別類存到 `save_<timestamp>/` 底下的子資料夾）、`check`（純語法檢查，不動系統）三個輔助子命令，且每個子命令都支援 `--dry-run`，可以只印出「會執行什麼指令」而不實際改動系統。

## 技術重點（實際讀過原始碼與版本紀錄）

- **版本演進清楚寫在腳本內建的 `HISTORY` 表裡**：從 2025-09-01 的純 Bash 版本，到 2025-10-04 改寫成 Python，到這次 v3.0.0（2025-10-16）新增「XinConf hardened, HTTPS/--insecure, auth, report.html」。也就是說這一版把 XinConf 狀態查詢從單純 `curl` 升級成支援 HTTPS 略過憑證驗證（`--insecure`）、Basic Auth（`--auth-user`/`--auth-pass`）、Bearer Token（`--auth-token`）。
- **`prep` 的完整鏈路**：偵測介面 IP → `hostnamectl set-hostname` → 用 regex 就地修補 `/etc/sysconfig/network/ifcfg-<iface>` 裡的 `DHCLIENT_SET_HOSTNAME`／`DHCLIENT_HOSTNAME` → 重啟網路 → 用「先移除同名舊行、再附加新行」的方式改寫 `/etc/hosts`（確保重跑不會疊出重複行）→ 正反解檢查 → 偵測 GDM 是否存在，有才切 `graphical.target` 並 enable GDM（沒裝 GNOME 的機器只會 WARN，不會硬切）→ 透過 curl（找不到 curl 才退回 `urllib`）打 XinConf 的 `checkstatus` API 拿狀態 → 把整個結果連同 Mermaid 流程圖一起渲染成一份深色卡片風格的 `report.html`。
- **附帶的 `DeBug_info/` 資料夾其實是這支工具的「手動版對照組」**：`wicked.txt` 裡是用 `sed` 直接改 `ifcfg-br0` 的 `DHCLIENT_SET_HOSTNAME`／`DHCLIENT_HOSTNAME`，`curl.sh` 是單純打 XinConf checkstatus 的裸指令、`hosts.txt` 是手動 `printf >> /etc/hosts` 的寫法——這三個片段分別對應到腳本裡的 `ensure_ifcfg_hostname()`、`fetch_xinconf_status()`、`write_hosts()`，剛好是「作者當初怎麼手動排查、後來怎麼寫成自動化函式」的第一手對照材料。
- **這份雲端硬碟上目前存放的 v3.0.0 檔案實際上有語法錯誤，跑不起來**：從 `render_report()` 開始（約第 384 行）之後所有函式本體都少了縮排，`REM ====` 分隔註解也漏打了 `#`（變成裸字串會直接觸發 NameError），連 `if __name__ == "__main__":` 都少打了底線變成 `if name == "main":`。實際用 `python3 -m py_compile sut_toolkit_albert_v3_0.py` 驗證，會直接丟出 `IndentationError: expected an indented block after function definition on line 384`。對照 `Previous/sut_toolkit_albert_v2_01.py` 可以正常編譯通過，代表這個縮排回歸是在 v2.0.1 之後、加入 report.html 與多種驗證方式這批改動時，疑似編輯器貼上時弄丟了縮排——這是實際跑過 `py_compile` 才確認的具體事實，不是腳本檔名或版本號能看出來的。

## 可延伸應用角度

- 可以做成一支「拆解一支寫壞的腳本」短片：直接示範 `python3 -m py_compile` 抓出這個 IndentationError，帶著看縮排從哪一行開始跑掉、`REM` 注解漏加 `#`、`__name__` 少打底線，當作一集迷你 code review／debug 素材，順便講「複製貼上大改版時最容易漏檢查的地方」。
- 也適合跟已經發過的 Health Guardian（`daily-tools/2026-08-20_health-guardian-cross-platform-toolkit`）搭成系列：兩支都是同一位作者的「Albert Style」腳本，共用同一套 PASS/INFO/WARN/FAIL/CRIT 色彩語意與深色 HTML 報表風格，一支負責「機器上機前的準備」（這支 SUT Toolkit），一支負責「機器上線後的健檢」（Health Guardian），剛好是同一套測試機生命週期的前後兩段，可以做一集「同一個人怎麼把重複的維運手工活都寫成同款工具」的系列比較。
- `--dry-run` 覆蓋所有子命令這件事，適合做成「安全示範一支會改系統設定的腳本」的教材切角——不用真的碰一台 SUSE 機器，就能完整展示 `prep` 的判斷邏輯與輸出格式。
