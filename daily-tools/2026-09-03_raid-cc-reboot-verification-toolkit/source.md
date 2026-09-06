# StartRAIDCC — 把「RAID Consistency Check 撐過 16 小時重開機驗證」拆成裝一次、關一次的四段式腳本

> 來源路徑：`StartRAIDCC/`（`install_start_cc.sh` / `start_cc.sh` / `check_cc.sh` / `monitor_cc.sh` / `cc_report.sh` / `uninstall_start_cc.sh` 為現行主線，`Maunl_Srcip/` 保留一支更早、單腳本自己觸發 reboot 的版本 `Reboot_CC_Test.sh` 與對應的 `Clean_Reboot_CC.sh`）
> 設計者：Albert.Chou（SIT / MiTAC Computing）

---

## 這是什麼

一組針對 MegaRAID controller（`storcli64`）的 **Consistency Check（CC）長時測試自動化腳本**，用在伺服器 RAID 驗證流程中：確保系統在超過 16 小時、經歷多次重開機的測試循環裡，CC 每次都會在開機後自動恢復執行，並把整段監控過程整理成一份可以直接拿給客戶看的彩色 HTML 報告。核心不是「呼叫 storcli 跑一次 CC」，而是把「裝服務 → 長時間輪詢狀態 → 出報告 → 驗證完一鍵清乾淨」這四個階段拆成四支各司其職的腳本，靠 systemd 服務讓 CC 自動恢復這件事不受「誰觸發了這次 reboot」影響。

## 為什麼值得關注

`Maunl_Srcip/` 資料夾裡留著一支更早的版本 `Reboot_CC_Test.sh`，做法是單一腳本自己扛下整個流程：等待開機後的緩衝時間、呼叫 `storcli /c0/vall start cc force`、檢查狀態、存 log、清舊 log，最後自己下 `reboot` 把機器重開——整個 16 小時的循環邏輯（`TOTAL_HOURS`、`CYCLE` 計數）都寫在同一支腳本的 while 迴圈裡。問題是這支腳本一旦執行到自己呼叫 `reboot`，bash 行程本身就被系統關機動作砍掉了，迴圈能不能繼續完全要依賴外部（systemd 服務）在下次開機後重新啟動它——但每次重啟腳本，`START_TIME`／`CYCLE` 都是從零重新計算，並沒有跨開機保留的狀態，16 小時的「總時長」在設計上其實沒辦法跨重開機正確累加。現行版本的做法是把「保證 CC 恢復」這件事單獨抽出來，用一個開機時觸發一次就結束的 `start_cc.service`（`Type=oneshot`）負責，不論這次重開機是腳本自己觸發、還是客戶的外部 reboot 工具觸發，開機後都會自動補跑 `storcli start cc force`；至於「跑多久、什麼時候該重開機」則完全交給 README 明講的「客戶提供的 reboot tools（不需自行寫 reboot script）」。等於把一個原本耦合在單一腳本裡、且有跨重開機狀態遺失風險的邏輯，拆成「不管誰重開機、CC 都會自己恢復」與「重開機這件事由外部工具負責排程」兩個互不依賴的關注點。

## 技術重點（實際看過下載下來的內容）

- **`install_start_cc.sh` 建的是 `Type=oneshot` 的 systemd 服務，不是常駐 daemon**：`start_cc.service` 只在每次開機時執行一次 `start_cc.sh`（呼叫 `storcli64 /c0/vall start cc force`），跑完就結束，靠 `systemctl enable` 保證「下次開機還會再跑一次」，而不是靠腳本自己維持一個長時間迴圈存活過 reboot。
- **`check_cc.sh` 已經設計了三種退出碼（0 / 2 / 3），對應 In Progress、Completed 視為通過（0），Stopped 視為需要人工介入的警告（2），無法判讀狀態視為失敗（3），但這個退出碼目前沒有被下游用上**——`monitor_cc.sh` 呼叫 `$CHECK_SCRIPT >> "$LOG_FILE" 2>&1` 只吃它的文字輸出寫進彙總 log，並不檢查離開碼；`cc_report.sh` 產報告時也是直接對 log 文字 `grep -q "\[PASS\]"` / `"\[FAIL\]"` / `"\[WARN\]"` 來分類，不是讀退出碼。三段退出碼比較像是先寫好、留給之後要做「單次檢查失敗就中止/告警」用途的擴充接口，目前的監控流程還沒接上它。
- **顏色標記怎麼從單次檢查一路傳到 HTML 報告**：`check_cc.sh` 的 `ok()/warn()/fail()` 把帶 ANSI 顏色碼、且夾著 `[PASS]`／`[WARN]`／`[FAIL]` 字樣的訊息印到 stdout；`monitor_cc.sh` 用 `>>` 把這些 stdout 原封不動接進自己的彙總 log；`cc_report.sh` 逐行讀這份 log，先用 `sed 's/\x1B\[[0-9;]*[A-Za-z]//g'` 把終端機 ANSI escape code 洗掉，再靠殘留的 `[PASS]/[FAIL]/[WARN]` 字樣分類上色，組成 HTML `<table>`——三支腳本之間沒有共用函式庫或設定檔，純粹靠「約定俗成的文字格式」串起來。
- **`monitor_cc.sh` 有自我修復：找不到已安裝的 `check_cc.sh` 就自己從當前目錄複製一份過去**：執行前會先檢查 `/usr/local/bin/check_cc.sh` 是否存在且可執行，不存在但當前目錄有 `./check_cc.sh` 的話，直接 `cp` 過去並 `chmod +x`，省掉 README 裡原本要求「先手動 `cp check_cc.sh /usr/local/bin/`」這一步，兩者算是有點重複但屬於防呆而非邏輯衝突。
- **舊版 `Maunl_Srcip/Reboot_CC_Test.sh` 額外整合了 event log 的存/清腳本**（`Save_event_log_v1.3.sh`、`Clear_event_log_v1.3.sh`，透過相對路徑呼叫，但這兩支本身不在這次下載範圍內），每個 reboot cycle 都會存一次系統事件 log、再清掉避免空間爆滿——這部分的「每輪存證＋清理」邏輯在現行拆分版本裡完全沒有對應功能，屬於重構時被拿掉、而不是被搬去別的腳本的能力。

## 可延伸應用角度

- **「單腳本自己觸發 reboot、狀態沒法跨重開機累加」→「拆成 oneshot 服務 + 外部排程」這個演進**，跟同一批素材裡 `SMBVIEW Memory Collector` 從「同次執行內硬做完製造模式切換」走到「拆兩階段、用 `/proc/stat` 的 `btime` 判斷是否真的重開過機」是同一種故事骨架，適合做成一支「reboot 相關的維運腳本，為什麼最後都會走向『拆開、用外部信號判斷狀態』」的系列影片，兩份素材可以互相印證同一個設計教訓。
- **`check_cc.sh` 設計了退出碼但下游沒用上**這一點，適合剪成「寫維運腳本時，先把退出碼分級設計好、之後再串接自動化判斷」的實務短片——現成的活教材：展示怎麼把 `monitor_cc.sh` 改成真的檢查 `check_cc.sh` 的退出碼，在偵測到狀態 2（Stopped）時主動告警，而不是事後才從 HTML 報告裡人工翻。
- **舊版被拿掉的「每輪存證 + 自動清理」能力**可以當作反面案例：討論重構把單體腳本拆開時，容易連帶遺失一些原本內建、但沒被明確列進新架構需求裡的周邊功能，適合搭配「重構前先列清單，逐項確認搬去哪」這類維運腳本重構方法論的主題。
