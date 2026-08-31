# GYM Pro v6.4.1 — 為了修一個 iOS 貼上匯入的老毛病，寧可砍掉一半功能重寫

> 來源路徑：`GYM/GYM Pro v6.4.1.html`（單檔 HTML+CSS+JS，262 行；對照 `Previous/GymProSystem20260123_merged_v631_fix.html` v6.3.1，697 行）
> 設計者：Albert Chou　最後更新：2026-03-20

---

## 這是什麼

一支單一 HTML 檔案就能跑的個人健身排程系統：每週課表、Tabata 間歇計時器（SVG 環形進度條）、運動項目新增/編輯/刪除、LocalStorage 本機儲存，外加一組給 iPhone Safari 用的資料匯入/匯出機制。沒有後端、沒有建置流程，開啟檔案就是完整應用。

## 為什麼值得關注

v6.4.1 不是一次功能加法更新，而是一次「為了修一個平台特有的 bug，寧可整支重寫並砍功能」的版本。README 裡記載的 v6.1.0 功能清單（依類別下拉篩選 Cardio/Strength、`allday` 全天時段、每個項目的 category badge、匯出時包一層 `meta`）在 v6.4.1 裡全部消失，換來的是一個新的 `btnSelectFile` + `<input type="file">` + `FileReader` 讀檔流程。這其實是在還原一個常見的 iOS Safari 踩坑：純文字貼上 JSON 匯入（`textarea` + 剪貼簿）在 iPhone 上常常因為長文字被系統截斷或貼上失敗而整包資料匯入失敗，作者選擇的解法不是修剪貼簿邏輯，而是繞開它——直接讀本機檔案。這對任何寫「純前端、給手機用」小工具的人是個具體案例：手機瀏覽器的限制有時候比多加一個功能更值得優先處理，即使代價是拿掉既有功能。

## 技術重點（實際看過程式碼）

- **`FileReader` 取代剪貼簿貼上**：`btnSelectFile` 觸發隱藏的 `<input type="file" accept=".json">`，選檔後用 `reader.readAsText(file)` + `reader.onload` 解析 JSON，而不是要求使用者手動複製貼上一大段文字進 `textarea`。程式碼裡保留了 `btnPasteImport`（文字貼上）作為備援，但把檔案讀取列為「iOS 推薦」選項，等於承認純貼上在 iOS 上不夠可靠。
- **匯入相容兩種資料格式**：`schedule = data.schedule || data`，同時可以吃 v6.3.1 那種包了 `meta` 外層的匯出檔，也可以吃直接就是 `schedule` 物件的舊格式，避免版本間匯入互不相容。
- **功能被砍到只剩核心**：v6.3.1 的 `type-filter`（類別篩選下拉）、`category-tag`（每個項目的類別徽章）、`allday` 全天時段、`edit-category`（新增/編輯時選類別）、匯出時的 `meta.exported_at/version/storage_key` 外層——這些在 v6.4.1 全部不見了，`schedule-container` 的渲染邏輯從 697 行縮到 262 行整支檔案。日期與時段只剩 `noon`/`night` 兩種，新增項目也不再要求選類別。
- **`STORAGE_KEY` 沒有跟著版號往前走**：檔名與畫面上都顯示 v6.4.1，但 `localStorage` 的 key 仍是 `gymProSchedule_v6_3`（v6.3.1 用的同一把 key），代表這次重寫刻意讓使用者從 v6.3.1 升級時舊資料能直接沿用，不用重新輸入課表——版號可以往前跳，但儲存格式的相容性刻意鎖住不變。
- **Tabata 計時器完全靠 `setInterval` + SVG `stroke-dashoffset` 手刻進度環**：沒有用 `requestAnimationFrame`，環形進度條的公式是 `circum - (tTime / total * circum)`，工作/休息切換時同步換色（`#00e5ff` ↔ `#ff4081`）而不是另外播動畫，邏輯與畫面更新綁在同一顆計時器裡。

## 可延伸應用角度

- **「加新功能前，先問這個裝置的原生限制修得動嗎」**這個決策過程本身就是好題材：多數教學只講怎麼加功能，這裡有個真實案例是「為了繞開 iOS 剪貼簿/長文字貼上的不穩定，寧可犧牲既有功能換穩定性」，適合做成「純前端小工具在 iPhone 上常踩的坑」系列。
- `STORAGE_KEY` 刻意不跟版號同步升級、只在資料結構真的不相容時才換 key 的做法，可以獨立做一支「LocalStorage 版本管理：什麼時候該換 key、什麼時候不該換」的短片，對照常見的「每次發版就換 key 導致使用者資料遺失」反例。
- 這支工具與同系列的 `Health Guardian`（`daily-tools/2026-08-20_health-guardian-cross-platform-toolkit`）一樣出自 Albert Chou、一樣是「單檔案零依賴」路線，可以比較「零依賴」在企業維運腳本與個人前端小工具兩種情境下分別解決了什麼問題。
