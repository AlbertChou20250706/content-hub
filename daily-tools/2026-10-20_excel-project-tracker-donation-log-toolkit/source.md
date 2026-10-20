`Excel_design_Sample_for_python_code` 是 Albert 的一組「辦公室生產力工具」合集：一支用 Python（`openpyxl`）一鍵產生多工作表 Excel 專案追蹤範本的產生器腳本，外加一套從 v2.3 一路迭代到 v4.1 的全離線捐贈紀錄網頁工具（純 HTML，不需伺服器也能輸出真正的 `.xlsx`/`.xls` 檔）。兩者本質都是同一個問題的不同解法：「怎麼讓 Excel 這種大家都會開、但很少人認真設計的表格,變成有結構、有驗證、能重複產生的工具」。

## 為什麼值得關注

工程團隊日常離不開 Excel——專案追蹤表、里程碑、參與者名單、捐款紀錄——但這些表格通常是手動複製上一份、欄位越改越亂、沒有下拉選單導致 Status 打成五種不同的字。這份素材示範了兩種截然不同但互補的解法：一種是「用程式碼定義範本，每次執行都重新生成乾淨的結構」（`make_albert_project_tracker.py`），另一種是「把表單介面搬到瀏覽器，離線也能用，存檔時才轉成真正的 Excel 格式」（Donation Sys 系列）。後者特別值得拆解的地方在於它橫跨了 v2.3 到 v4.1 六個以上版本，可以看到一個人在沒有外部需求方壓著的情況下，怎麼自己一版一版把離線匯出的相容性問題解決掉。

## 技術重點

實際看過下載下來的腳本與網頁原始碼後，幾個設計細節值得注意：

- **`make_albert_project_tracker.py` 用資料驅動定義 9 張工作表**：`SHEETS_SPEC` 字典一次列好 `Open_Items`、`Closed_Items`、`Participants`、`Milestones`、`RAID_Card_Definition`、`PVT_Label_Format`、`FTP_Info`、`Decisions_Statement`、`Dashboard_Readme` 的欄位，標題列統一橘色底、粗體、置中並自動換行，且每張表都 `freeze_panes` 凍結首列,欄寬依標題文字長度自動計算（最小 14）。
- **用 `DataValidation` 在既有儲存格範圍內塞下拉選單**：Status（Open/In Progress/Blocked/Closed/On Hold）、Priority（Critical/High/Medium/Low）、Risk_Level 等欄位在第 2～2000 列全部套上資料驗證，避免使用者手動輸入造成同一個狀態出現多種拼法。`Milestones` 工作表還內建 `=IFERROR(Actual_Date-Planned_Date,"")` 公式自動算出 Slip_Days,兩個日期只要有一個沒填就安全地顯示空白而不是錯誤值。
- **Donation Sys 的離線匯出走「雙保險」策略**：v3.6 開始預設匯出單一 `.xlsx`（內建 SheetJS,`現金捐贈者`／`物品捐贈者`兩個工作表),但只在網路可用、SheetJS 能被載入時才生效;同時保留兩份獨立 `.xls` 檔案的匯出路徑作為備援,離線時也保證能開啟,`.xls` 版本還做了表頭底色、2px 邊框與數字千分位這些列印友善的細節。
- **現金與物資分開統計、估值欄位可選擇是否落地**：物資捐贈多了「估值 (NT$)」欄位,但這欄預設只參與前端統計與圖表,使用者要勾選「匯出包含物資估值」才會真的寫進檔案——這是刻意把「畫面上看到的數字」跟「落地存檔的數字」分開控制的設計,避免估值這種主觀數字被誤當成正式紀錄的一部分。

## 可延伸應用角度

- 適合剪一支「用 Python 把公司常用 Excel 範本變成可重複生成的工具」的影片,示範 `openpyxl` 的 `DataValidation` 跟凍結窗格這類常被忽略但很實用的 API,跟之前的 SIT 入職追蹤表（VBA 版）、OT Calculator 這類辦公室小工具系列互相呼應。
- Donation Sys 從 v2.3 到 v4.1 的版本演進,很適合做成「一個離線網頁工具怎麼一步步解決『存檔相容性』問題」的系列,重點放在「雙重匯出保險」這個思路本身,可以延伸到任何需要離線優先、但又要跟 Excel 互通的場景。
- 兩個工具可以合併成一支「同一個人怎麼用兩種完全不同的技術路線（後端腳本生成 vs. 前端離線網頁）解決同一類表格管理問題」的對比影片,討論什麼情境該選哪一種。
