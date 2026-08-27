# AI 股市週報自動化：GitHub Actions 觸發 + Claude 生成 + LINE 群組推播規劃

> 自動化工具／規劃階段：用 GitHub（類似 cowork 的排程／手動觸發行為）驅動 AI 產生股市週報，並推播到 LINE 群組

| 項目 | 內容 |
|---|---|
| 日期 | 2026-08-27 |
| 分類 | 自動化工具 |
| 狀態 | 草稿 |
| YouTube | |
| NotebookLM | |

## 背景

想做一個「每週自動產生 AI 股市週報，並發到 LINE 群組」的小工具，觸發方式希望借用 GitHub 的排程／手動觸發能力（`schedule` cron + `workflow_dispatch`），行為上類似目前用 Claude Code Cloud／cowork 那種「雲端上有東西在跑、隨時可以手動補跑一次」的體感，而不是自己顧一台常駐主機。

這篇是**規劃階段**的技術紀錄：先把架構、流程、待確認事項寫清楚，實作程式碼會另外開一個獨立 repo（依本 repo `CLAUDE.md` 的內容規範，content-hub 只放 Markdown、不放程式碼／CI 工程邏輯），這裡只做飛輪的第一步——把技術規劃寫成文字紀錄。

**強制規範（不可省略）**：只要是這個自動化產出、對外發送或發布的任何內容（LINE 訊息、YouTube 說明欄、social post），都必須固定附上這句免責聲明：

> 【投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議】

這句話要直接寫進 AI 生成週報的 prompt 樣板裡，當作每次輸出的固定收尾，不是靠人工每次手動加。

## 做了什麼（規劃中的架構）

目前規劃的完整流程，分成「觸發 → 資料 → AI 生成 → 格式化 → 發送 → 存檔」六段：

1. **觸發層**：GitHub Actions
   - `schedule`（cron）：例如每週一台灣時間 08:00 自動跑（cron 以 UTC 記錄，需自行換算時差）
   - `workflow_dispatch`：手動觸發，對應「類似 cowork」的臨時補跑／測試需求
   - 之後可視需求再加 `repository_dispatch`，讓外部系統主動叫它跑
2. **資料蒐集層**：抓大盤指數、漲跌幅、成交量、產業族群表現（候選資料源：FinMind、yfinance、TWSE OpenAPI），視需求再加財經新聞標題，整理成結構化 JSON
3. **AI 生成層**：把結構化資料包進 prompt，呼叫 Claude API 生成週報文字（市場總結、產業亮點、風險提示、下週觀察重點），prompt 中固定嵌入上述免責聲明
4. **格式化層**：把生成文字轉成 LINE 訊息格式（純文字＋emoji，或用 LINE Flex Message 做卡片式排版）
5. **發送層**：呼叫 **LINE Messaging API** 的 push message 到指定群組
   - 注意：**LINE Notify 已於 2025 年 3 月停止服務**，不能再用；要改用 Messaging API 的 Bot channel
   - 前置作業：Bot 先被邀入目標群組，透過一次 webhook 事件取得 Group ID（只需做一次）
6. **存檔層**：每次生成結果存成檔案（例如 `reports/YYYY-MM-DD.md`）commit 回自動化 repo，作為歷史紀錄與除錯依據；失敗時也發一則 LINE 訊息通知自己

以上步驟目前都還沒有實際程式碼，屬於規劃內容，實作會在另一個獨立 repo 進行。細部設計（資料 schema、prompt 規則、GitHub Actions workflow 草案、錯誤處理、安全與成本評估）見 [`notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md`](notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md)。

## 關鍵發現／重點

- **LINE Notify 已停止服務**，這個方向一開始就要排除，直接規劃 LINE Messaging API + Bot channel + Group ID 的路線，避免走回頭路。
- **免責聲明是固定成本，不是選配**：使用者明確要求「投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議」必須出現在所有對外輸出（LINE 訊息、影片說明、social post），最保險的做法是寫進 AI 生成的 prompt 樣板，讓它變成每次輸出的固定結尾，而不是依賴人工事後補加。
- **免責聲明要求 prompt 裡「逐字輸出、不可改寫」**：只交代「請附上免責聲明」不夠，LLM 有機率意譯換句話說，導致文字跟使用者要求的固定句子不一致。
- **LINE 訊息格式分兩階段做**：先上純文字版本把整條流程跑通，穩定後再優化成 Flex Message 卡片，不用一開始就求視覺完美。
- **自動化程式碼與內容紀錄要分兩個 repo**：GitHub Actions workflow、資料抓取腳本、LLM 呼叫邏輯屬於工程檔案，依 `CLAUDE.md` 規範不能放進 content-hub；content-hub 只負責事後把「怎麼做」寫成技術紀錄。
- **免費資源的限制要提前抓好**：LINE Messaging API 免費方案每月訊息則數有上限、GitHub Actions（private repo）有分鐘數額度，週報一週一次的頻率下應該都在免費額度內，但要在正式上線前確認清楚。

## 行動項（若有後續待辦）

- [x] LINE Bot（Messaging API channel）已存在，直接沿用既有的「ChouAP.Cloud」channel 與已核發的 Channel Access Token，不需另外申請
- [ ] 另開一個獨立的自動化 repo（工程程式碼與 CI 設定都放這裡，不放 content-hub）
- [ ] 選定股市資料來源並驗證資料品質（候選：FinMind／yfinance／TWSE OpenAPI）
- [ ] 定義結構化資料 JSON schema（大盤指數／產業族群漲跌／選配個股與新聞標題）
- [ ] 設計 Claude API 生成週報的 prompt 樣板，固定內嵌免責聲明並要求逐字輸出
- [ ] 先實作純文字版 LINE 訊息，穩定後再評估是否做 Flex Message 卡片
- [ ] 設定 GitHub Actions：`schedule` + `workflow_dispatch`，每個步驟拆開寫（不要用 `&&` 串長鏈），並用 GitHub Secrets 管理金鑰
- [ ] 加上失敗重試（1-2 次）與失敗通知機制
- [ ] 確認 LINE Messaging API 免費方案的月訊息則數上限
- [ ] **開發／測試階段先用「自己與 Bot 的一對一聊天」驗證 push message**（用個人 User ID，不用 Group ID），避免測試雜訊發到正式群組
- [ ] 整條流程（資料→生成→格式化→發送→存檔）在一對一測試環境跑穩定後，才邀請 Bot 加入正式的兩個目標群組（股市資訊資訊通、ETF討論區），取得對應 Group ID 並切換為正式版本
- [ ] 實測正式群組發送無誤，穩定跑過幾週後再回來這裡錄 YouTube 教學、回填發布狀態

## 延伸資源

- 相關 repo：（自動化實作 repo，待建立，尚無連結）
- 相關文章／前作：本 repo [ChouAP.Cloud 上雲 SOP](../2026-08-12_chouap-cloud-migration-sop/README.md)（Cloud environment 建置方式與 Setup script 拆行寫法可參考）
- 參考資料：LINE Messaging API 官方文件、LINE Notify 服務停止公告
- 完整細部設計：[`notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md`](notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md)

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
