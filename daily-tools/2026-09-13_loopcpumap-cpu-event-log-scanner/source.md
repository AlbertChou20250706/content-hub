# LoopCpuMap（Albert Style）— 只讀 log 就能把「第幾圈」對上「CPU 被誰調過」的比對工具

LoopCpuMap 是一支純粹掃描既有 log 檔的比對工具：它不連線、不讀取即時系統狀態，只靠正則表達式在一堆測試留下的 txt/log 裡，同時抓出兩件事——「現在跑到第幾圈（Loop/Round/Iteration/Cycle）」以及「CPU 在這段時間有沒有被調頻、被限頻、或直接出錯」，然後把兩者用檔名、行號、log 裡的時間字串對在一起，輸出 Markdown、HTML、CSV 三種報表。

## 為什麼值得關注

長時間跑測（burn-in、可靠度測試、老化測試）最常見的爭議是：「效能下降/當機到底發生在第幾圈？」而現場往往只留下一堆 `dmesg`、`messages_since_last_boot.txt`、`seloutput.txt`、BMC SEL 這類雜訊很重的 log，人工翻找非常痛苦。這支工具的定位很窄但很實用：不做任何監控代理、不需要在受測機上裝東西，測試跑完之後才把 log 丟進去做「事後追溯」，剛好卡在很多重量級監控方案（需要常駐 agent、需要即時遙測）不划算的那個空隙裡。

## 技術重點

專案有兩個版本，直接體現了一個工具從「堪用」到「順手」的演進：

- **v1.0（`LoopCpuMap_v1.0_Albert.py`，322 行）**：純 CLI，腳本本身硬編碼一組 console 色彩/圖示（INFO/PASS/WARN/FAIL/HEAD 對應 ANSI 顏色），並在 Windows 上手動呼叫 `kernel32.SetConsoleMode` 開啟 ANSI escape 支援，這樣純 Python 腳本在 cmd.exe 裡也能有顏色輸出。掃描完直接把三種報表丟到 `./Logs`。
- **v1.1（`LoopCpuMap_v1.1.py`，410 行）**：在 CLI 之上包了一層 Tkinter GUI（可切 Light/Dark 主題），並且把「圈數判斷」跟「CPU 事件判斷」的規則全部開放成可編輯欄位——log 檔名 glob pattern、圈數正則、TUNE/THROTTLE/ERROR 三類關鍵字都能在 GUI 裡改，跑完除了三份報表還多存一份 `LoopCpuMap_config_<timestamp>.json` 把當次設定原樣記錄下來，方便回頭對照「這次到底用什麼規則掃的」。同時保留 `--no-gui` 純 CLI 模式，適合排進自動化流程。

判斷邏輯本身很直白但夠用：
- 圈數用正則 `(?i)(^|[^a-z])(Loop|Round|Iter(?:ation)?|Cycle)[^0-9]{0,10}(\d{1,6})` 抓，能吃 `Loop 12`、`Round: 3`、`Iteration-99`、`Cycle 7` 這幾種寫法。
- CPU 事件分三桶：**TUNE**（`governor`、`cpufreq`、`scaling_max_freq`、`intel_pstate`、`HWP` 等主動調整訊號）、**THROTTLE**（`PROCHOT`、`Power Limit`、`Current limit`、`VRM`、`Thermal` 等硬體強制降頻訊號）、**ERROR**（`MCE`、`RCU stalled`、`lockup`、`TSC unstable` 等系統級異常）。
- 時間對齊只是「抓出同一行裡看起來像時間戳的字串」（`YYYY-MM-DD HH:MM:SS`、syslog 式 `Oct  7 14:03:59`、或 kernel 的 `[ 1234.5678]`），README 裡明講不做真正的時間轉換，純粹留給人工肉眼比對——這是刻意的取捨，換取零依賴、免設定就能跑。
- 預設掃描的檔名清單很貼近伺服器測試現場：`messages_since_last_boot.txt`、`dmesgOutput.txt`、`seloutput.txt`、`sensorOutput.txt`、`iRMC_*SEL*`、`g2t_testCaseRun*.txt` 等，最後保底再吃一次全部 `*.txt`。

## 可延伸應用角度

適合放進「跟 SIT/老化測試現場除錯有關」的系列裡，跟先前已收錄的 kdump 健檢腳本、C-State 待機功耗記錄工具組放在一起講「測試現場排錯三件套」——一個抓當機留證、一個量測待機功耗異常、這個則是把「第幾圈」跟「CPU 被誰調過」兜起來。也可以單獨做一支「log 考古」主題的影片：示範同一批 log 丟進 v1.0 跟 v1.1，對照純 CLI 與 GUI 版本在同一份 log 上抓出一致結果，順便講解為什麼作者選擇「只做字串比對、不做即時監控」這個设计取捨。
