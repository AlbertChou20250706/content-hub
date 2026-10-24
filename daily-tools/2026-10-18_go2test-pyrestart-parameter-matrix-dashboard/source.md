go2test 參數對照表是把 **go2test / PyRestart** 測試框架裡 Stress、Reboot、DC、AC、WOL、RunAll 等各種測試情境所需的工具、時間、RSCID、負載、SCC 設定全部彙整成一張查詢表，再分別做成 CSV、分組上色的 Markdown、互動式 HTML 儀表板三種輸出格式，解決「每次要組 go2test 指令都要回頭問人或翻舊紀錄」的問題。

## 為什麼值得關注

go2test / PyRestart 這類內部測試框架最大的痛點不是工具本身難用，而是**參數組合的知識散落在人的腦中**：哪個 Group 要搭配哪個 RSCID、SCCI 不可用時要補哪個 `--sccdisable` 代碼、VMware guest 要多加哪個旗標——這些慣例通常只存在於熟悉的工程師記憶裡，新人或換手時只能照抄舊指令、出錯了也不知道為什麼。這份資料夾把這些隱性知識攤開成結構化的 CSV（88 列）＋文件＋可互動儀表板三層呈現，而且版本從 v1.0 一路做到 v1.2，顯示這是持續被回頭補強的活文件，不是寫一次就扔著的筆記。

## 技術重點

實際下載並讀過資料夾內容後，重點如下：

- **CSV 是單一事實來源**：`Group, ToolID, Owner, Tool, Profile, Duration, RSCID, Load, SCCDisable, RSC, Options, Notes` 十二個欄位，把「這次測試用哪個工具、誰負責、跑多久、要停用哪些 SCC 代碼」全部結構化記錄，v1.0 版 88 列，另有重建後的精簡版 32 列。
- **已知問題的慣例補丁被明文寫下**：SCCI 在 Go2Test 裡目前不可用，預設要在 PyRestart 執行時加 `--sccdisable=202` 當緩解；VMware guest 要加 `--noagentcheck`（必要時 `--sccstop`）——這種「已知限制 + 標準繞過方式」的配對，正是最容易在交接時遺漏的細節。
- **互動儀表板讓查表變成點選＋組指令**：v1.2 的 `go2test_parameters_dashboard_v1.2_albert.html` 用純前端 JS（`fillGroups`／`renderParams`／`compose`／`toCSV`）做到依 Group 篩選、即時組出可直接貼上執行的 go2test 指令字串、並能把目前篩選結果重新輸出成 CSV，全程不需後端或安裝任何套件。
- **連錯字修正都記錄在案**：`PwerCtrl`→`PowerCtrl`、`OTLIS`→`OLTIS`、`DEFAUTL.key`→`DEFAULT.key` 這類在原始資料裡出現過的拼字錯誤，也被整理進文件，避免未來整理資料時把錯字當成正確欄位名稱繼續複製下去。
- **debug.sh 是個小而實用的旁支**：三行指令片段，從 PTU（Power/Thermal Utility）的執行紀錄裡抓前 200 行、再用 `grep -A3 -B3` 找 `Launching PTU`／`error`／`usage` 關鍵字，附帶提醒要先 `sed -i 's/\r$//'` 處理 CRLF 換行再執行，是典型「從真實除錯需求中長出來」的小工具。
- 文件本身誠實標註「這是資料與文件資料夾，不是可執行的工具」——它的價值不在程式碼本身，而在把分散的測試知識結構化、版本化、可追溯。

## 可延伸應用角度

- 適合做成「把隱性知識變成結構化文件」系列：示範如何把團隊裡只存在對話與記憶裡的測試慣例（像 SCCI 緩解、VMware 旗標）萃取成 CSV＋互動儀表板，降低交接與新人上手的成本。
- 也適合搭配其他測試框架（如同樣用 RSC 服務、PyRestart 的 Reboot/AC/DC 循環測試）做成「同一套 RSC 測試基礎設施，不同工具怎麼共用參數慣例」的技術比較。
- debug.sh 的「三行抓重點」模式可以單獨拉出來，講解如何用最小的 grep/sed 組合快速定位長執行紀錄裡的關鍵訊息，適合資訊量不大但很實用的短片素材。
