# AI 股市週報自動化技術規劃（NotebookLM 原始素材）

> 這份文件是給 NotebookLM 直接餵入用的技術規劃原稿，內容比 README.md 更詳細一些，之後若錄製 YouTube 逐字稿產出，可視情況保留本檔作為原始素材存檔，或由逐字稿取代。

## 目標

打造一個「每週自動產生 AI 股市週報，並推播到 LINE 群組」的自動化流程，觸發方式借用 GitHub 的排程／手動觸發能力，體感類似 Claude Code Cloud／cowork 那種「雲端上有東西在跑、隨時可以手動補跑一次」的工作模式。

## 整體架構（六段流程）

```
① 觸發 → ② 資料蒐集 → ③ AI 生成 → ④ 格式化 → ⑤ LINE 發送 → ⑥ 存檔
```

### ① 觸發層

- `schedule`（cron）：例如每週一台灣時間 08:00 自動觸發。cron 表達式以 UTC 記錄，台灣是 UTC+8，換算時記得處理跨日問題。
- `workflow_dispatch`：手動觸發按鈕，對應「類似 cowork」的臨時補跑／除錯需求。
- 選配：`repository_dispatch`，讓外部系統（另一支腳本、webhook）主動叫它跑。

### ② 資料蒐集層

- 抓大盤指數、個股／族群漲跌幅、成交量等結構化資料。
- 候選資料源：FinMind、yfinance、TWSE OpenAPI（依實際可用性與資料品質篩選）。
- 視需求加財經新聞標題（RSS 或新聞 API）。
- 輸出格式：結構化 JSON，作為 AI 生成的輸入素材。

### ③ AI 生成層

- 把結構化資料包進 prompt，呼叫 Claude API。
- 生成內容涵蓋：本週市場總結、產業族群亮點、風險提示、下週觀察重點。
- **固定規則**：prompt 樣板中必須內嵌以下免責聲明，作為每次輸出的固定收尾：

  > 【投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議】

  這句話不是靠人工事後補加，而是寫死在 prompt 裡，確保每一次自動生成的內容都一定包含。

### ④ 格式化層

- 把生成文字轉成適合 LINE 的訊息格式：可以是純文字＋emoji，也可以進階用 LINE Flex Message 做卡片式排版（標題、重點條列、免責聲明分區塊呈現）。

### ⑤ 發送層（LINE Messaging API）

- **重要前提**：LINE Notify 已於 2025 年 3 月停止服務，不能再用這條路線。
- 改用 **LINE Messaging API**：
  1. 申請 Bot（Messaging API channel），取得 Channel Access Token。
  2. 把 Bot 加入目標 LINE 群組。
  3. 透過一次 webhook 事件取得該群組的 Group ID（只需做一次，之後重複使用）。
  4. 用 push message API，帶 Channel Access Token + Group ID + 訊息內容，推播到群組。
- 留意免費方案每月訊息則數上限，週報一週一次的頻率下應該足夠，但正式上線前要實際確認額度。

### ⑥ 存檔層

- 每次生成結果存成檔案（例如 `reports/YYYY-MM-DD.md`），commit 回自動化 repo，作為歷史紀錄與除錯依據。
- 若當次流程失敗（資料源異常、API 呼叫失敗等），額外發一則 LINE 訊息通知自己，方便及時人工介入補跑。

## 細部設計（第二輪規劃）

以下是六段流程往下拆到可執行層級的細節，供之後實作時直接參考。

### 資料層 Schema 設計

週報用的結構化資料建議固定成以下 JSON 骨架，欄位寧可先少、之後再加：

- 日期區間：本週一至週五收盤
- 大盤指數：加權指數、櫃買指數的開盤／收盤／漲跌幅
- 產業族群：漲幅 Top 5、跌幅 Top 5
- （選配）成交值排行前 20 大個股
- （選配）3-5 則財經頭條標題

### Prompt 設計規則

- **System prompt**：明確定位「週報撰寫助手，不是投資顧問」，避免 LLM 自行延伸出建議性語氣。
- **User prompt**：帶入上述 JSON，要求輸出固定五段：市場總結 → 產業亮點 → 風險提示 → 下週觀察重點 → 免責聲明。
- **免責聲明必須指定「逐字輸出、不可改寫」**：只寫「請附上免責聲明」不夠，LLM 有機率會意譯、換句話說，導致最終文字跟使用者要求的固定句子不一致。要在 prompt 裡明確要求原句照抄：

  > 【投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議】

### LINE 訊息格式（分兩階段）

1. **第一階段（先上線）**：純文字訊息＋emoji，先把整條流程跑通、驗證穩定。
2. **第二階段（穩定後優化）**：LINE Flex Message 卡片，標題區／內容區／免責聲明區分開呈現，視覺更好。不需要一開始就做。

### GitHub Actions Workflow 架構草案

以下為實作時的參考草案（放在獨立自動化 repo，不是 content-hub）：

```yaml
name: weekly-stock-report
on:
  schedule:
    - cron: '0 0 * * 1'   # UTC 週一 00:00 = 台灣週一 08:00
  workflow_dispatch: {}
jobs:
  generate-and-send:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: python fetch_data.py
      - env: { ANTHROPIC_API_KEY: "${{ secrets.ANTHROPIC_API_KEY }}" }
        run: python generate_report.py
      - env:
          LINE_CHANNEL_ACCESS_TOKEN: "${{ secrets.LINE_CHANNEL_ACCESS_TOKEN }}"
          LINE_GROUP_ID: "${{ secrets.LINE_GROUP_ID }}"
        run: python send_line.py
      - run: git add reports/ && git commit -m "weekly report" && git push
      - if: failure()
        run: python notify_failure.py
```

**重要**：每個安裝／執行步驟拆成獨立一行，不要用 `&&` 串成一條長鏈。這是 [ChouAP.Cloud 上雲 SOP](../../2026-08-12_chouap-cloud-migration-sop/README.md) 那篇已經踩過的雷——一步失敗、外層又有容錯把錯誤吞掉，會讓 Setup script／workflow 誤判成功，其實後面步驟根本沒執行。

### 錯誤處理與重試

- 資料抓取／LLM 呼叫失敗時允許重試 1-2 次（簡單的 exponential backoff）。
- 任何步驟真的失敗就讓 workflow fail，用 `if: failure()` 額外發一則 LINE 訊息通知自己，不要讓錯誤靜默過去。

### 安全與治理

- 所有金鑰（Claude API Key、LINE Channel Access Token）只放 GitHub Secrets，不寫死在程式碼裡。
- Group ID 雖非高敏感資訊，但建議一併放 Secrets，避免 hardcode 外洩被騷擾。
- 留意 LINE Channel Access Token 是否為長效 token，定期檢查有效期並視需要更新。

### 成本與額度評估

- **GitHub Actions**：一週一次的用量，public repo 免費、private repo 額度也綽綽有餘。
- **Claude API**：一週一次呼叫，token 用量小，成本可忽略。
- **LINE Messaging API**：免費方案每月訊息則數有上限，一週一次遠低於門檻，但正式上線前建議實際查一次目前方案的確切數字。

## 架構分工原則

- **自動化程式碼（GitHub Actions workflow、資料抓取腳本、LLM 呼叫邏輯）放在獨立的自動化 repo**，不放進 content-hub。這是因為 content-hub 的 `CLAUDE.md` 明確規定只放 Markdown 內容，禁止放程式碼／CI 工程邏輯（`.github/workflows/` 例外僅限於儲存庫生命週期的 LINE 通知，不含內容生成或爬蟲邏輯）。
- **content-hub 只負責事後把「怎麼設計、怎麼踩坑」寫成技術紀錄**，走「新增主題」SOP，之後視進度錄成 YouTube 教學、轉成 NotebookLM 音訊。

## 待確認事項（實作前）

- 股市資料源的實際可用性、更新頻率、是否有使用條款限制。
- LINE Messaging API 免費方案的月訊息則數上限，是否足夠長期使用。
- GitHub Actions（若自動化 repo 為 private）的分鐘數額度是否足夠。
- Prompt 樣板的免責聲明位置與格式，需要在正式上線前實際測試生成結果，確認每次輸出都確實包含。
- 失敗通知機制的具體設計（例如失敗幾次後改用其他管道通知）。

## 免責聲明（強制規範，逐字保留）

> 投資一定有風險，基金/ETF/股票投資有賺有賠，以上資訊非投資建議

此句必須出現在：AI 生成週報的 LINE 訊息內容中、YouTube 影片說明欄、以及所有相關 social post。
