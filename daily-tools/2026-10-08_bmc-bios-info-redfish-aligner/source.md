`get_bmc_bios_info.sh` 是一支單檔 Bash（內嵌 Python）腳本，透過 Redfish API 以唯讀方式讀取伺服器 BMC 與 BIOS 資訊，自動比對並修正 BIOS 自動化測試用設定檔（`check_system_config.sh`）裡過時或寫錯的屬性名稱，取代過去逐項打開 BIOS 畫面人工核對的作法。

這支腳本其實是「BMC Auto Test 套件（pg_20250902）」這個更大的 SUT 端自動化測試包其中一支，README 完整說明了整套工具（環境安裝、韌體更新、壓力測試、FRU 備份等十幾支腳本），但這次 Drive 資料夾裡實際只放了 `get_bmc_bios_info.sh` 本體與 README，其餘腳本並未一併提供，下面的技術重點只針對這支已下載的腳本。

## 為什麼值得關注

BIOS 自動化測項的設定檔（`check_system_config.sh`）裡要填一堆 `bios_*` 屬性名稱，但同一個 BIOS 畫面項目在不同專案、不同 BIOS 版本之間名稱會漂移——可能這版叫 `NMI Function`，下一版改叫 `NMI Button`，而 BMC Redfish 底層的識別碼又是完全不可讀的 `OBDEV5010` 這種代碼。過去要核對這些名稱，得真的連進 BMC 網頁、一條條跟設定檔比對，在要跑多台、多機型 SIT 驗證機台的場景下非常耗工又容易漏看。這支腳本把「讀取 BMC 現況 → 比對設定檔 → 找出哪裡對不上 → 自動修正並輸出新檔」整條流程自動化，而且全程只對 BMC 做 GET，不會意外改到任何 BIOS 設定，是一個「風險很低但省下大量人工核對時間」的典型自動化案例。

## 技術重點

實際下載並讀過腳本本體（1315 行）與 README 後，重點如下：

- **密碼不落地**：互動輸入密碼不回顯，透過 `mktemp` 建立的暫存 curl config 檔（`chmod 600`，腳本結束用 `trap` 刪除）傳給 `curl -K`，密碼不會出現在 `ps` 清單或任何存檔裡；只有 BMC IP 與帳號會記到 `~/.get_bmc_bios_info.last`，方便下次直接按 Enter 沿用。
- **名稱比對演算法是核心**：比對 `check_system_config.sh` 裡的 `bios_*` 名稱與 BMC 屬性時，忽略空格、底線、大小寫差異（`Number of PSU` = `Number_of_PSU` = `number of psu`），同名但分屬不同通道的項目（例如 COM1 與 EMS 都叫 `Console Redirection`）則用腳本設定裡的「群組」欄位（`bios_items` 的最後一欄）去對應到正確的 BMC 屬性，避免抓錯。
- **screen / attr 雙模式**：`config_name_mode` 預設 `screen`，寫回設定檔時用「BIOS 畫面上看到的名稱」（這是因為下游測試系統認這個命名，名稱帶空格會直接判 fail）；也可切成 `attr` 模式直接寫 Redfish 屬性名稱，供 `change_bios_setting.sh` 之類會用屬性名稱當 key 讀寫 BIOS 的腳本使用。
- **版本號自動推算 N / N-1**：讀到 BMC 實際執行中的韌體版本後，會自動判斷寫入 `bmc_n_revision`／`bmc_n1_revision`（BIOS 同理），並嘗試從版本號反推上一版（`4.06` → `4.05`）；但次版本剛好是 `00` 時（例如 `7.00`）沒辦法用算的，會標 WARN 請人工確認，不會硬猜一個可能錯的值。連韌體映像檔檔名裡的版本號也會依照命名規則一併改寫。
- **只改需要改的行**：產生的 `check_system_config_<timestamp>.sh` 採取精準 diff 寫入，沒問題的行（含註解、縮排）原樣保留，被修正的名稱行會順便補上 `#BIOS: <選單路徑>` 註解，讓人看 diff 就知道改了什麼、為什麼改。
- **五級狀態判讀**：PASS / WARN / FAIL / ERROR / N/A，各自對應明確情境（例如「對到不只一個屬性」是 FAIL、「此 BMC 沒有該項目」是 N/A），報表每個章節標題旁還會顯示該類問題數量的小膠囊，方便先看哪裡要處理。
- **三種輸出同時產生**：給人看的 HTML 報表、給程式解析的 CSV（`Section,Item,Status,Detail`）、以及 `logs/raw/*.json` 原始 Redfish 回應備查，三者用同一組 timestamp 對齊，方便事後追查某個欄位的原始數據。

## 可延伸應用角度

- 很適合做成「自動化測試設定如何應付屬性命名漂移」的教學案例——這是跨版本、跨專案自動化驗證裡很常見但很少被公開拆解的真實痛點，這支腳本的比對演算法剛好是一個完整、可讀的參考實作。
- 技術重點裡「screen name vs 底層 attribute name 雙模式比對」可以單獨抽出來做一段簡短技術分享，概念可以套用到任何「人看得懂的名稱」與「機器用的識別碼」需要互相對應的場景，不限於 BMC/BIOS。
- 可以和既有的 SIT 系列工具（如先前介紹過的 NIC/RDMA 健康監控、SIT 統一指揮中心等）串成「BMC/BIOS 自動化驗證生態系」系列，因為這些工具原本就同屬同一套 SUT 端自動化測試基礎設施。
