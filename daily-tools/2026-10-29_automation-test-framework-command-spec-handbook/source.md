「Automation new VM已建置完畢」資料夾收錄的是一份跨兩個版本（v1.8.2、v7_0306）的《Automation Testing Software Operating Users Guide》——支撐內部一整套硬體測試自動化框架（Auto Test software v1.0.1）的指令定義與操作規格書，本身不含程式碼，是一份 SOP／操作手冊。

## 為什麼值得關注

這個候選在 2026-09-28 第一次掃到時只有兩份 docx、沒有任何 README，無法判斷它究竟是工具、純文件還是 SOP，因此被暫緩收錄；直到 2026-10-28 Albert 補上一份 README，明確標註「類型：SOP（操作步驟）與使用者操作指引文件，無程式碼，不是可執行的工具」，才確認可以安心介紹——這也是本系列第一次收錄「規格書」而非「工具」類型的素材。

真正讓它值得一讀的是內容本身的規模與維護紀錄：手冊的 Change List 從 0408 到 0801 之後持續有多位工程師（Sherry.kao、Caby.chen、Kenny.li、Rosa.chen）各自新增、修改測試關鍵字，涵蓋近 30 項測試能力（機器人控制盒、BMC 內部測試、Prime95、MAC 比對、SDR/Threshold 事件測試等），說明這不是一次性寫完就沒人再碰的文件，而是一套被團隊持續擴充的測試框架背後的「指令字典」。

## 技術重點

實際展開兩份 docx 的內容後，幾個具體設計值得記錄：

- **機器人控制盒（Robot control box）有一套明確的指令 DSL**：格式類似 `ctrl_box <ctrl_box_ip> ac <box_id> <slot_id> <ac_channel_id> ac_on`，涵蓋 AC 電源開關（含延遲時間）、前面板按鍵模擬（power/reset/id/nmi button，且區分 `ing`／`finish` 兩種等待語意——前者先回傳再等待、後者等按鍵動作結束才回傳）、LAN／USB 線材動態插拔，以及 LED 狀態檢查（on/off、blinking、顏色、閃爍頻率、計數器，頻率容許誤差標註為 ±5ms）。
- **BMC 內部測試工具以「SDR 實測值 vs. FW spec」為核心比對邏輯**：`auto_sdr_test_tool` 會檢查 BMC 回報的 sensor list 是否與 FW spec 的 sensor list 一致，分別處理 Threshold Sensor（含 Redfish path、sensor owner identity、unit type、AC/DC 開機狀態下是否該有 SDR 資料）與 Discrete Sensor 兩種類型，且針對不同 PSU 供應商各自維護獨立的門檻值表，找出 redundant／missing sensor 時會明確列出清單而非只回報「比對失敗」。
- **涵蓋面橫跨電源、溫度、韌體、網路與 BMC**：同一套框架底下還包含 Prime95 壓力測試自動化、`mac_compare`、`dcmi Power Reading`、event log filter、SOL（Serial-over-LAN）連線控制、C-State CSV 解析，以及 `auto_redfish_threshold_test_tool`，顯示這是一套以指令關鍵字驅動、可被測試案例逐步串接的端到端硬體驗證腳本庫。

## 可延伸應用角度

- 比起整份手冊的總覽介紹，更適合挑一個具體切角做深讀短片——例如單獨拆解機器人控制盒的指令 DSL 設計，或是 BMC SDR 與 FW spec 比對這段「如何用規則取代人工目視判讀感測器門檻值」的邏輯。
- 可以跟先前介紹過的 BMC/BIOS Redfish 比對工具（bmc-bios-info-redfish-aligner）、Prime95 壓力工具（prime95-cpu-stress-toolkit）串成系列，呈現同一套硬體自動化測試生態系裡，規格文件與實際腳本工具是如何對應起來的。
- 這份素材本身是 SOP 而非程式碼，也適合做一支「如何快速讀懂一份測試框架規格書」的教學短片，示範怎麼從章節架構（設備需求 → 指令定義 → 預期結果判定 → 控制關鍵字）快速定位到自己需要的測試步驟，而不用整份從頭看到尾。
