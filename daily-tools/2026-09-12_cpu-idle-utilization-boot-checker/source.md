# CPU Idle Utilization Boot Checker（開機閒置 CPU 使用率檢查工具）

一支專門檢查「伺服器開機完成後，CPU 是否真的回到閒置狀態」的自我安裝型 Shell 腳本組，由一支安裝腳本與一支解除安裝腳本構成，核心用途是攔截開機後殘留背景程序、驅動輪詢、韌體代理程式吃滿某顆核心的異常情況。

## 這是什麼

`check_cpu_idle_detail.sh` 本身不是最終的檢查程式，而是一支「安裝器」：執行後會先確認 `mpstat`（來自 `sysstat` 套件）是否存在、不存在就用 `yum install -y sysstat` 補齊，接著用 heredoc 把真正的檢查邏輯寫死產生到 `/usr/local/bin/check_cpu_idle_detail.sh`，再把這支新產生的腳本掛進 `/etc/rc.d/rc.local`，讓它在「每次開機」都會自動背景執行一次。配套的 `uninstall_cpu_idle_check.sh` 則負責乾淨反安裝：刪除 `/usr/local/bin` 下的腳本、從 `rc.local` 移除該行、並清掉 `/var/log/cpu_idle_check.log`。

## 為什麼值得關注

伺服器 SIT（System Integration Test）最常見的一類回歸問題，是「重開機之後某顆核心一直卡在高使用率」——可能是某個 kernel module 初始化沒收斂、某個 daemon 卡在忙等迴圈、或韌體代理程式輪詢間隔設太短。這類問題如果只看整機平均 CPU 使用率很容易被稀釋掉（32 核心裡有 1 核心跑滿，平均值可能還在 3% 以下），必須逐核心（per-core）檢查才抓得到。這支工具的價值正是把這件事自動化、常態化，並且做成開機即掛勾的形式——不需要人工在每次重開機後手動盯著 `top` 或 `mpstat` 看。這跟同一批來源裡的 `raid-cc-reboot-verification-toolkit`、`kdump-helper-crashkernel-toolkit` 屬於同一類「開機後自動驗證」的測試腳手架，都是圍繞「reboot 之後系統狀態是否正常」這個測試場景在建構。

## 技術重點（實際讀過原始碼）

- **兩層腳本設計**：外層是「安裝器」，內層（heredoc 產生的內容）才是真正每次開機執行的邏輯。這種「安裝腳本產生執行腳本」的寫法讓部署變成單一檔案即可完成，不用另外準備第二個檔案上傳。
- **逐核心判斷，不看整機平均**：用 `mpstat -P ALL 1 1` 抓一次所有核心的統計，`grep -E "^[0-9]"` 過濾出各核心那幾行，再用 `awk` 取出核心編號（第 3 欄）與 idle 值（第 13 欄），用 `100 - idle` 換算成使用率。
- **睡 30 秒才量測**：內層腳本一開頭先 `sleep 30`，等系統開機初始化的暫時性 CPU 尖峰過去，避免把「正常的開機瞬間負載」誤判成異常，這是一個容易被忽略但很實務的細節。
- **閾值判斷（THRESHOLD=5）**：任何一顆核心使用率超過 5% 就標記 `FAIL_FLAG=1`，並在 log 裡用 ❌ 標出是哪顆核心超標，最後統一輸出 PASS / FAIL 結論到 `cpu_idle_check.log`。
- **開機自動掛勾**：用 `grep -q ... /etc/rc.d/rc.local` 判斷是否已經掛過，避免重複安裝時寫入重複的啟動行，再 `echo ... >> /etc/rc.d/rc.local &` 背景執行，不阻塞開機流程。
- **對稱的乾淨反安裝**：`uninstall_cpu_idle_check.sh` 對每一個安裝步驟都有對應的清除動作（腳本檔、rc.local 那一行、log 檔），三個步驟都有 `if [ -f ... ]` / `if grep -q ...` 的存在性檢查再動作，不會因為重複執行而報錯。

## 可延伸應用角度

- 適合剪成「伺服器開機健康檢查腳手架」系列的一集，跟 `raid-cc-reboot-verification-toolkit`、`kdump-helper-crashkernel-toolkit` 放在同一個系列脈絡裡對照講——同樣是「重開機後自動驗證」，但這支專注在 CPU 閒置率、另外兩支分別驗證 RAID 卡狀態與 crashkernel 設定，三支放在一起可以講「一套完整的開機回歸測試組合」是怎麼堆出來的。
- 也適合單獨拉出來講「用 `sleep 30` 迴避開機瞬間尖峰」這個測試設計細節，延伸成一集講「寫自動化硬體測試腳本時，時間點抓不對會怎麼誤判」的實務案例分享。
