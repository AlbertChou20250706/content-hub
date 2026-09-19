CallbackTool 是一套完全建構在 Excel 365 Power Query 上的 WMS（倉儲管理系統）CSV 自動驗證工具：使用者每天從 WMS 匯出兩份庫存清單（Onhand Query、Item Query）覆蓋進固定資料夾，打開一份 `callback_check.xlsx` 按【全部重新整理】，就能自動檢查每一筆料號（Item Number，須為 12 碼）與客戶序號（Customer SN，須為 10 碼）是否符合規則，並產出 Leader 專用的異常清單與統計報表——全程不寫一行 VBA。

## 為什麼值得關注

這是一個「用 Excel 原生功能取代腳本」的典型案例：多數人遇到「每天要對兩份 CSV 做格式檢查、篩異常、給主管報表」這種需求，直覺會寫 Python 或 VBA，但作者選擇完全用 Power Query 的 M 語言搭建整套 pipeline，好處是接手的人不需要會寫程式、不需要裝額外環境，只要會用 Excel 就能維護。更值得注意的是它把「可搬移、可交接」當成第一設計目標：透過一個 `tblConfig` 表格用公式自動抓取工作簿所在路徑，讓整個工具資料夾複製到任何磁碟機、任何電腦都不用改一行設定就能用，README 裡甚至寫了一段「搬移測試」章節來驗證這件事。對於需要把工具交給非工程背景同事長期使用的場景，這種「零程式碼、路徑自動化、有明確驗收標準」的設計思路很有參考價值。

## 技術重點

實際下載並讀過 `README_v8.md`、`callback_check.xlsx` 以及兩份真實資料（`Onhand_latest.csv` 3,195 筆、`ItemQuery_latest.csv` 4,425 筆，皆為 RX4770M8 專案的真實料號與客戶序號資料）之後，重點如下：

- **用 Excel 表格模擬「設定檔」**：`Config` 工作表用 `=LEFT(CELL("filename",A1),FIND("[",CELL("filename",A1))-1)` 這條公式取得目前工作簿所在資料夾路徑，再包成一個叫 `tblConfig` 的表格，讓所有 Power Query 查詢都能用 `Excel.CurrentWorkbook(){[Name="tblConfig"]}[Content]{0}[FolderPath]` 動態取路徑——這是不寫 VBA 也能做到「工具整包搬家免改路徑」的關鍵技巧。
- **M Code 直接把資料驗證邏輯寫進查詢**：附錄完整列出五段 M code，核心是在載入 CSV 後用 `Table.AddColumn` 疊加 `ItemLen`／`SNLen`（用 `Text.Length` 算長度)，再用 `Text.Combine` + `List.RemoveNulls` 組出一句可讀的 `Issue` 說明（例如「Item Number Length<>12」），最後用 `IsAbnormal` 布林欄位標記異常列——用純函數式的鏈式寫法做完整套資料品質檢查，是很紮實的 Power Query M 語言範例。
- **明確處理 Excel 對長數字字串的「科學記號地雷」**：README 特別強調 Customer SN 這種 10 碼數字若被 Excel 誤判成數值格式會變成 `2.4E+09`，因此在 M code 裡用 `Table.TransformColumnTypes` 強制轉成 `type text` 並 `Trim`，同時在操作規範上明文禁止「直接雙擊開 CSV」，把一個 Excel 老手都會踩的坑寫成強制流程規範。
- **異常查詢用「查詢疊查詢」而非重新讀檔**：`Check_Onhand` / `Check_ItemQuery` 兩個查詢直接 `Source = Onhand_latest` 引用前一個查詢的輸出再 `Table.SelectRows` 篩選 `IsAbnormal = true`，`Summary` 查詢又再引用這兩個查詢做統計——三層查詢疊加、只讀一次原始檔，改動規則只需要修改最底層查詢，其餘會自動連動更新。
- **文件本身就是一份可驗收的 SOP**：README 不只是說明書，每個建置步驟後面都附「驗證點」（例如「工作表是綠色表格，不是普通範圍」「Customer SN 不會變成科學記號」），並提供一組真實資料下的預期輸出（Onhand 3,195 筆中 1,252 筆異常、ItemQuery 4,425 筆中 1,351 筆異常）當作驗收基準，新手照著做完可以直接對答案。

## 可延伸應用角度

- 適合做成「不寫程式也能做資料驗證自動化」系列的示範案例，對比之前介紹過的 Python／Shell 類工具，凸顯 Power Query M 語言在「輕量、免安裝、交給非工程背景同事維護」場景下的優勢與限制。
- 「用 CELL 公式反推工作簿路徑、包成表格給 Power Query 讀」這個技巧本身值得獨立拉一段出來，是很多 Excel 老手都不知道、但能解決「共用檔案路徑寫死」痛點的實用招式。
- 可以搭配 `README_ToolDB-紀錄CIP`（同樣是 Excel／VBA 類工具）組成一集「當 Excel 也能做資料工程」的比較集，分析什麼情境該用巨集、什麼情境該用 Power Query、什麼情境該直接上腳本語言。
