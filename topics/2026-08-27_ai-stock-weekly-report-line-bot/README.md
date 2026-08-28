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

這篇是技術規劃紀錄：架構、流程、待確認事項寫在這裡，實作程式碼放在另一個獨立 repo（依本 repo `CLAUDE.md` 的內容規範，content-hub 只放 Markdown、不放程式碼／CI 工程邏輯）——[`ai-stock-weekly-report-bot`](https://github.com/AlbertChou20250706/ai-stock-weekly-report-bot)，目前已完成骨架（資料抓取、prompt 樣板、LINE 推播、GitHub Actions workflow），處於個人 LINE 帳號測試階段。

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

以上六段流程已在 [`ai-stock-weekly-report-bot`](https://github.com/AlbertChou20250706/ai-stock-weekly-report-bot) 完成骨架實作（資料源選定 yfinance，尚未接上正式群組）。細部設計（資料 schema、prompt 規則、GitHub Actions workflow 草案、錯誤處理、安全與成本評估）見 [`notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md`](notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md)。

## 關鍵發現／重點

- **LINE Notify 已停止服務**，這個方向一開始就要排除，直接規劃 LINE Messaging API + Bot channel + Group ID 的路線，避免走回頭路。
- **免責聲明是固定成本，不是選配**：使用者明確要求「投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議」必須出現在所有對外輸出（LINE 訊息、影片說明、social post），最保險的做法是寫進 AI 生成的 prompt 樣板，讓它變成每次輸出的固定結尾，而不是依賴人工事後補加。
- **免責聲明要求 prompt 裡「逐字輸出、不可改寫」**：只交代「請附上免責聲明」不夠，LLM 有機率意譯換句話說，導致文字跟使用者要求的固定句子不一致。
- **LINE 訊息格式分兩階段做**：先上純文字版本把整條流程跑通，穩定後再優化成 Flex Message 卡片；三條自動化目前都已完成第二階段（見下方「視覺升級」）。
- **自動化程式碼與內容紀錄要分兩個 repo**：GitHub Actions workflow、資料抓取腳本、LLM 呼叫邏輯屬於工程檔案，依 `CLAUDE.md` 規範不能放進 content-hub；content-hub 只負責事後把「怎麼做」寫成技術紀錄。
- **免費資源的限制要提前抓好**：LINE Messaging API 免費方案每月訊息則數有上限、GitHub Actions（private repo）有分鐘數額度，週報一週一次的頻率下應該都在免費額度內，但要在正式上線前確認清楚。
- **`.TW` 後綴會被 LINE 誤判成網域名稱**：純文字訊息裡寫「3706.TW」會被自動轉成一個點了會壞掉的連結（`.TW` 是真實的國別頂級域名），三個 repo 的股票代號顯示都改成去掉交易所後綴。
- **Claude API 的 web_search 允許網域清單，網域可能突然連不上**：`allowed_domains` 裡只要有一個網域當下對 Anthropic 的爬蟲不可達，整個請求會直接 400 失敗（不是只有那個網域的搜尋失敗），實測就在美股週報上踩到（`marketwatch.com`／`reuters.com`／`wsj.com` 一度回報不可達）。三個 repo 的週報／委員會生成程式都加了「retry without web_search」的防呆，網域失效時報告仍會照常送出（只是少了新聞連結），不會整篇開天窗。
- **法人買賣超等真實數據要注意發布時機，不是查不到就代表程式有 bug**：TWSE 三大法人 T86 報表要等收盤後才會公布（約台灣時間下午 3 點後），在交易時段內手動測試會查到「沒有符合條件的資料」，屬預期行為而非程式錯誤；程式端已設計成查無資料就整段省略，不會編造數字。

## 行動項（若有後續待辦）

- [x] LINE Bot（Messaging API channel）已存在，直接沿用既有的「ChouAP.Cloud」channel 與已核發的 Channel Access Token，不需另外申請
- [x] 個人一對一測試（`curl`／PowerShell 手動 push）已驗證 Bot → 個人這條路可行；過程中一度在終端機截圖裡不慎露出 token，已重新核發並更新 `content-hub` 的 GitHub Secret
- [x] 另開獨立自動化 repo：[`ai-stock-weekly-report-bot`](https://github.com/AlbertChou20250706/ai-stock-weekly-report-bot)（Public）
- [x] 選定股市資料來源：yfinance（`config/watchlist.json` 可自行調整觀察清單，免改程式碼）
- [x] 定義結構化資料 JSON schema 並實作 `src/fetch_data.py`
- [x] 設計 Claude API 生成週報的 prompt 樣板（`prompts/system_prompt.md`），固定內嵌免責聲明並要求逐字輸出，程式碼端（`src/generate_report.py`）也加了一層防呆：若模型輸出漏掉免責聲明會自動補上
- [x] 實作純文字版 LINE 推播（`src/send_line.py`），Flex Message 卡片留待穩定後再評估
- [x] 設定 GitHub Actions `weekly-stock-report.yml`：`schedule` + `workflow_dispatch`，每個步驟拆開寫，金鑰走 GitHub Secrets
- [x] 加上失敗通知機制（`src/notify_failure.py`）；失敗重試目前只有 Claude SDK 內建的自動重試，`fetch_data.py`／`send_line.py` 尚未加自訂重試邏輯
- [x] 在 `ai-stock-weekly-report-bot` repo 設定 GitHub Secrets（`ANTHROPIC_API_KEY`、`LINE_CHANNEL_ACCESS_TOKEN`、`LINE_PUSH_TARGET_IDS`＝個人 User ID），手動觸發 `workflow_dispatch` 端對端測試成功（資料抓取→Claude生成→LINE推播→自動存檔全部跑通）
- [x] 新增美股週報（`ai-stock-weekly-report-bot`，S&P500／那斯達克／道瓊＋必看 NVDA、TSM ADR），與台股週報共用同一支 `send_line.py`
- [x] 視覺升級：台股週報、美股週報、委員會報告三條自動化都改成 **LINE Flex Message 卡片 + 真實 K 線圖**（`mplfinance` 依 yfinance 實際 OHLC 資料繪製，非 AI 畫圖），並依可信度限制 `web_search` 只能引用官方／主流財經網站
- [x] 台股週報、委員會報告加上真實三大法人（外資／投信／自營）買賣超表格，直接查 TWSE 官方 T86 端點取得，非模型估算；查無資料就整段省略
- [x] 修掉 `web_search` 允許網域清單一旦有網域連不上就整篇報告開天窗的問題：加上「retry without web_search」防呆，三條自動化都已套用並實測通過
- [ ] 確認 LINE Messaging API 免費方案的月訊息則數上限
- [ ] 整條流程在一對一測試環境跑穩定後，才邀請 Bot 加入正式的兩個目標群組（股市資訊資訊通、ETF討論區），取得對應 Group ID 並把 `LINE_PUSH_TARGET_IDS` 切換為正式版本
- [ ] 實測正式群組發送無誤，穩定跑過幾週後再回來這裡錄 YouTube 教學、回填發布狀態

## 排程總覽（三條自動化，台灣時間依序發送）

| 時間 | Repo | 內容 |
|---|---|---|
| 06:00 | `ai-stock-weekly-report-bot` | 台股週報 |
| 07:00 | `stock-committee-bot` | 3706／00935／009816 委員會深度分析（3則） |
| 07:30 | `ai-stock-weekly-report-bot` | 美股週報（S&P500／那斯達克／道瓊＋必看 NVDA、TSM ADR） |
| 每月 1 日 08:00 | `quality-picks-bot` | 台股／美股優質個股＋ETF 前 3 名推薦 |

> 測試期間前三個排程暫時都改成**每天**發送（cron 每日觸發），方便快速抓問題；待使用者確認沒有其他狀況後，再改回原定的每週一。`quality-picks-bot` 因為看的是變化緩慢的基本面數據，一開始就設計成每月一次。

## 衍生專案：AI 股市投資決策委員會（資料驅動版）

在週報之外，另外把使用者過去手動設計的「三代理人委員會」提示詞產生器（`committee_prompt_generator.html`，手動複製貼到 claude.ai 那種）搬上雲端，獨立開一個 repo：[`stock-committee-bot`](https://github.com/AlbertChou20250706/stock-committee-bot)。

- 必看代號（3706 神達、00935 野村臺灣新科技50、009816 凱基台灣TOP50）與比較組寫在 `config/must_watch.json`，可擴充
- 關鍵差異：RS 動能報酬率、止損價、部位規模這些數字**改由程式碼實際抓歷史股價計算**，不再讓 Claude 憑空估算，Claude 只負責論述與決策文字
- 原本的手動 HTML 工具保留在該 repo 的 `manual-tool/` 下，作為臨時查任意標的用的 ad hoc 工具
- 跟 `ai-stock-weekly-report-bot` 共用同一個 LINE Bot（ChouAP.Cloud）與 Claude API key 概念，各自獨立的 repo／Secrets
- 每個標的一則獨立的 LINE Flex Message，內含真實 K 線圖（該標的近三個月走勢）、RS 濾網結果、比較組相對強弱、量化風控數據表，以及查 TWSE T86 端點取得的三大法人買賣超（查無資料就整段省略，不編造）

## 衍生專案二：優質標的推薦（quality-picks-bot）

前兩個自動化看的都是**價格動能**（週漲跌、RS 動能），使用者後續加碼希望再有一份「台股／美股優質個股＋ETF 推薦」，看的是**基本面品質**，因此又獨立開一個 repo：[`quality-picks-bot`](https://github.com/AlbertChou20250706/quality-picks-bot)。

- 候選池：台股個股／台股 ETF／美股個股／美股 ETF 各 6～8 檔知名龍頭與主流 ETF（非全市場掃描，免費 API＋GitHub Actions 排程架構下不現實），程式依量化公式排序取前 3 名
- 篩選公式：個股＝0.35×ROE + 0.25×營益率(或毛利率) + 0.20×自由現金流為正 + 0.20×股利品質；ETF＝0.5×資產規模 + 0.5×(1－費用率)，每項先在候選池內做標準化，缺資料的欄位得分視為 0（不用平均值猜測，避免獎勵缺資料的標的）
- 排序結果由程式決定，Claude 只負責引用程式算好的數字說明「為什麼夠格」，不能重新排序、不能自己推薦排序外的標的
- 不涵蓋共同基金：Sharpe 值／Alpha／最大回撤／4433 排名目前沒有已驗證的免費資料源，暫不做
- 首次上線實測（2026-08-28）確認 yfinance 對候選池內的台股／美股標的財報數據（ROE、毛利率、自由現金流、ETF 費用率／規模）涵蓋度良好，數字與已知公開資訊量級相符（如台積電 ROE ≈ 40%、Mastercard ROE 因高額庫藏股導致帳面淨值極低而異常偏高等，均為真實財報特性、非資料異常）

## 延伸資源

- 相關 repo：[`ai-stock-weekly-report-bot`](https://github.com/AlbertChou20250706/ai-stock-weekly-report-bot)（週報自動化）、[`stock-committee-bot`](https://github.com/AlbertChou20250706/stock-committee-bot)（委員會深度分析自動化）、[`quality-picks-bot`](https://github.com/AlbertChou20250706/quality-picks-bot)（優質標的推薦，每月一次）
- 相關文章／前作：本 repo [ChouAP.Cloud 上雲 SOP](../2026-08-12_chouap-cloud-migration-sop/README.md)（Cloud environment 建置方式與 Setup script 拆行寫法可參考）
- 參考資料：LINE Messaging API 官方文件、LINE Notify 服務停止公告
- 完整細部設計：[`notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md`](notebooklm_sources/AI_STOCK_WEEKLY_REPORT_LINE_BOT_PLAN.md)

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
