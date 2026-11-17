`LogCheckTool` 資料夾裡其實放了同一個問題的兩套獨立解法：一支 Windows PowerShell 腳本（`LogCheckTool_v1.6.5r2.ps1`）遞迴掃描資料夾裡的 `.log`/`.txt`，找出含關鍵字的行並輸出 CSV + HTML 報表；另一支是純前端單檔 HTML 工具「Log Scanner Pro」（`LogScanner_Albert_v2_3.html`），用瀏覽器的 Stream API 在不上傳任何檔案的前提下掃描數十 GB 的 log。兩者都是 Albert.Chou 的作品，分別對應「在機台上直接跑腳本」與「把整個 log 資料夾丟進瀏覽器分析」兩種不同的使用情境。

## 為什麼值得關注

SIT/測試現場撈到一堆 log 之後,人工翻找 `ERROR`/`FAIL`/`CRITICAL` 這類關鍵字是最浪費時間但又最容易漏看的步驟,尤其當 log 數量上百、單檔又動輒幾百 MB 時,純靠 Ctrl+F 或丟給同事肉眼巡一輪根本不現實。這兩支工具剛好補了兩個不同的使用場景的洞:PowerShell 版適合直接放在待檢查的機台或 log 根目錄上跑,不需要額外安裝任何東西;瀏覽器版則解決了「log 太大、太多,想在自己電腦上分析但不想上傳到任何伺服器」的資安顧慮——Log Scanner Pro 的 README 特別強調全程 Client-Side,檔案不會被送出。兩套工具並存也反映了同一個人在不同階段針對同一個痛點的兩次獨立嘗試。

## 技術重點

實際下載並讀過兩支程式本體(PS1 共 200 行、HTML 單檔共 614 行)與各自 README 後,重點如下:

- **PowerShell 版用單一正規表示式比對**:`$Keywords` 陣列(預設含 `FAIL`、`ERROR`、`WARNING`、`event`、`CRITICAL`,也包含专案特定字串如 `sel-log.*`、`--rscid=*`)用 `-join '|'` 拼成一條 regex 丟給 `Select-String -CaseSensitive:$false`,一次掃描同時比對所有關鍵字,比逐個關鍵字掃描快很多。
- **PS1 版會自動跳過雜訊**:遞迴掃描時主動排除 `.zip`/`.tar`/`.xz` 壓縮檔與路徑含 `sosreport` 的檔案,並各自計入「略過數」顯示在最終摘要,避免把一堆解壓縮產物或系統診斷包誤判成一般 log。
- **HTML 報表有筆數防呆**:PS1 版輸出的 HTML 最多只顯示前 10,000 列(`$MaxHtmlRows`),超過會在摘要標 `(TRUNCATED)` 並提示完整結果在 CSV 裡——這是因為表格渲染筆數太多時瀏覽器會卡死,用「先給摘要、重資料查 CSV」的方式取得平衡。
- **瀏覽器版的防卡死核心是真的做了時間切片**:程式裡用 `performance.now()` 量測已經處理的時間,一旦超過 `YIELD_MS` 門檻就 `await new Promise(r => setTimeout(r, 0))` 把主執行緒讓出去,再搭配 `TextDecoderStream` 做串流讀取,這樣掃描大檔案時分頁不會凍結——這跟「把整個檔案讀進記憶體再處理」是完全不同的設計思路,也是能處理 50GB+ log 而不崩潰的關鍵。
- **瀏覽器版用 `webkitdirectory` 取得整個資料夾**:使用者點「選擇 Log 資料夾」時,是透過 `<input type="file" webkitdirectory directory multiple>` 一次性取得資料夾下所有檔案的控制代碼,搭配只篩選 `.log`/`.txt`/`.csv`/`.xml`/`.json`/`.out`/`.err` 副檔名與自動跳過 0 bytes 空檔,減少不必要的 I/O。
- **匯出格式特別照顧中文與 Excel**:CSV 匯出內建 `﻿` BOM,確保 Excel 開啟中文內容不會變成亂碼,並把「檔名」與「完整路徑」分成兩欄,比單一路徑欄位更方便在 Excel 裡篩選排序。
- **兩個版本各自記錄了明確的疑難排解經驗**:PS1 版 README 寫了「HTML template not found 就要把範本檔放到同一資料夾」這類真實踩過的坑;HTML 版的 Changelog 則記錄了 v2.1→v2.3 的三次迭代(Time-Slicing 核心、容量計算修正、CSV 編碼優化),顯示這不是寫完就沒再動過的一次性工具。

## 可延伸應用角度

- 適合做成「同一個問題,兩種架構」的對比教學:PowerShell 腳本代表「在目標機台本地執行、檔案系統直接存取」的傳統思路,瀏覽器版則代表「零安裝、純前端、靠瀏覽器新 API 處理大檔案」的現代思路,兩者的取捨(部署方式、效能上限、資安考量)是很好的架構選擇案例。
- Log Scanner Pro 的 Anti-Freeze 時間切片設計(`performance.now()` + 定時讓出主執行緒)可以單獨抽出來做一段「如何讓瀏覽器處理超大檔案不卡死」的技術短片,這個模式能套用到任何需要在前端處理大量資料的場景,不限於 log 分析。
- 可以跟之前介紹過的其他 SIT 維運小工具(如 LoopCpuMap、SaveLog/LogPurge)串成「現場故障排查工具箱」系列,主題聚焦在「怎麼從一堆雜亂的機台輸出裡,快速揪出真正的異常」。
