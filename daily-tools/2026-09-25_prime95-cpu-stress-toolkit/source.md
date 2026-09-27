Prime95 Interactive Stress Tool（Albert Style）是一支包在 `stress_interactive.sh` 裡的互動式 Bash 控制腳本，把 mprime（Prime95 Linux 版）包裝成一套可直接在 RHEL 8.x / 9.x 上跑的系統整合測試（SIT）壓力工具，跑完自動產出深色風格 HTML 報告。

## 為什麼值得關注

CPU 壓力測試在 SIT 現場是天天要跑的例行工作，但原生 mprime 只會丟出一坨純文字 log，PASS/FAIL 要工程師自己盯著看關鍵字，測試中途按 Ctrl+C 中斷也常常什麼報告都留不下。這支工具把「選時間、選模式、跑測試、判讀結果、產報告」整段流程收斂成一個互動選單，而且中斷防護做得很完整——即使手動中斷也會補產報告並標記 INTERRUPTED，不會斷尾。從 README 裡完整的 v1.0.0 到 v1.6.0 版本紀錄可以看出，這也是一支被持續使用、持續被回饋改進的實戰工具，不是寫完就丟著的一次性腳本。

## 技術重點

實際下載並讀過 `stress_interactive.sh` 原始碼後，重點如下：

- **三種壓測模式對應真實硬體驗證目的**：Small FFTs 測 CPU L1/L2 Cache 發熱極限、Large FFTs 測記憶體控制器與 RAM 穩定性、Blend Mode（預設）做系統整體穩定性測試。腳本用 `declare -A MODE_MAP` 把選單編號對應到模式名稱與 mprime 的 `TortureTest` 參數值，再寫進 `local.txt` 讓 mprime 依此執行。
- **trap-safe 報告產生機制（v1.5.0 起）**：用 `trap on_interrupt INT TERM` 攔截 Ctrl+C／TERM，並用 `trap generate_report EXIT` 確保無論正常結束或被中斷，`generate_report()` 都一定會被呼叫一次（用 `REPORT_DONE` 旗標防止重複執行）。中斷時的 `INTERRUPTED` 旗標會讓最終結果直接標成橘色的 `INTERRUPTED`，跟 PASS（綠）／FAIL（紅）明確區分。
- **PASS/FAIL 判讀邏輯**：先從 log 裡用 `grep -oE` 統計 `errors`／`warnings` 出現次數加總，再搭配關鍵字比對（`FATAL`、`hardware failure`、`mismatch`、`TORTURE TEST FAILED`）綜合判定，不是只看單一字串就下結論。
- **HTML 報告是自己刻的深色主題模板**，統一套用「SIT Master Toolkit」視覺風格：頂部大字 RESULT 橫幅（v1.6.0 新增）、Workers Completed / Total Errors / Total Warnings 統計表、系統硬體快照（CPU 型號、核心數、記憶體、Kernel 版本，靠 `lscpu`／`free -h`／`uname -r` 即時抓取並用 `: "${SYS_CPU:=Unknown}"` 做空值 fallback），完整原始 log 收進可折疊的 `<details>` 區塊，需要時再展開。
- **`timeout` 做硬性時間保護**：`timeout "$STRESS_TIME" "$MPRIME_BIN" -t | tee -a "$RESULT_LOG"` 一行同時做到時間到強制終止、log 落地跟即時輸出。

## 可延伸應用角度

- 適合做成「Bash 腳本強化教學」系列：以 v1.0.0 到 v1.6.0 的版本紀錄為主軸，拆解 trap-safe 設計、helper function 抽出（`say`/`die`/`rule`/`html_escape`）這些從「能動」進化到「工程化」的重構過程。
- 可搭配既有的 SIT 系列工具內容（例如已發布的 OT 計算器、NIC/RDMA 監看工具）一起討論「Albert Style 報告引擎」這個跨工具共用的深色 HTML 報告視覺規範，講統一風格對維運團隊的價值。
- 也適合做「中斷不斷尾」這個具體技術點的短影片：對比一般腳本被 Ctrl+C 中斷後什麼都不留 vs. 這支工具用 trap 機制仍能補產報告，是很好的 defensive scripting 案例。
