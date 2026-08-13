# 三大原創系列 — 內容生產工作流手冊

> 本文件是 content-hub 的「系列策展層」，記錄三個原創系列各自的定位，
> 以及一支新影片從錄音到上架、歸檔的完整跨 repo 流程。
>
> 管線本身的技術細節（Whisper 參數、Profile 判斷邏輯、AST 驗證等）
> 以 `audio_transcribe` repo 的 `README.md` 為準，本文件不重複，
> 只做跨 repo 銜接與系列策展的說明。

---

## 系列與 Profile 對照表

| 系列 | audio_transcribe Profile | content-hub 系列資料夾 | 播放清單 |
|---|---|---|---|
| AI機槽成長歷程 | `trend`（⚠️ 待確認，見下方） | `series/ai-rack-growth/` | <補上連結> |
| DSOX4024G 示波器實戰系列 | `oscilloscope` | `series/dsox4024g-oscilloscope/` | <補上連結> |
| Claude Code 雲端開發實戰系列 | `aidev` | `series/claude-code-cloud-dev/` | <補上連結> |

> ⚠️ 「AI機槽成長歷程」對應的 Profile 是依內容領域推測（伺服器/AI 硬體
> 趨勢，跟 `trend` 的描述最接近），麻煩確認一下實際錄製這批影片時
> 用的是哪個 profile 前綴，錯了幫我更正這張表。

---

## 一支新影片，從錄音到歸檔的完整流程

### 第一階段：錄音到影片上架（在 `audio_transcribe` repo）

依系列選對應的 profile 前綴命名錄音檔，接著照 `audio_transcribe` 的
`README.md` **STEP 1-9** 走完整套：轉錄 → 校對 → handoff → NotebookLM
→ 影片回爐轉字幕 → 字幕校對 → 章節生成 → YouTube 上架。

**幾個容易忘記的檢查點**（都是實戰踩過的坑，別再摔第二次）：

- [ ] STEP 5.5 幫影片改檔名時，**前綴必須跟原始錄音完全一致**，不能
      用 NotebookLM 自動產生的檔名——否則會被 Stage 0 判定成全新
      主題，同一系列的內容會被拆成兩個互不相通的 profile
- [ ] YouTube 描述貼上前，肉眼確認**沒有殘留 `<<>>` 或 `【】`**——
      這兩種都是 handoff 樣板的佔位符或分類標籤，不該出現在發布內容裡
- [ ] 章節時間戳第一行必須是 `0:00`，YouTube 才會啟用章節條
- [ ] 瀏覽權限先選「不公開」，STEP 9 驗收清單過一輪確認沒問題，
      才改成「公開」
- [ ] 取消勾選「設為即時首播影片」——首播模式下無法先預覽

### 第二階段：加入播放清單

上架後，把影片加進對應系列的 YouTube 播放清單（上表「播放清單」欄）。
如果是該系列的第一支影片，先建立播放清單（瀏覽權限公開、語言繁中），
命名比照現有系列風格（例如「XX實戰系列」「XX成長歷程」）。

### 第三階段：回填 content-hub（GitHub 上的 SEO 錨點）

1. 在 `topics/` 底下建立這支影片的 topic 資料夾（`YYYY-MM-DD_slug/`），
   照 `_templates/` 起手，補上 `README.md`、`youtube_metadata.md`、
   `social_posts.md`
2. 在對應的 `series/<slug>/README.md`「目前集數」表格新增一列
3. 更新 `_log/publish_log.md`

### 第四階段：分開 commit

`content-hub` 這邊的異動（topic 資料夾、series README）直接 commit +
push。`audio_transcribe` 那邊的異動（profile／glossary 更新）**另外
commit + push，兩個 repo 分開推送**，不要混在同一筆。

---

## 系列資料夾骨架（新增第四個系列時參考）

```
series/<slug>/
└── README.md   ← 系列定位、目前集數表、內容方向、相關檔案位置
```

新增系列時，複製 `claude-code-cloud-dev/README.md` 的格式起手即可，
不用另外做 template（三個系列的量還不到需要模板化的程度，先手動維護，
等量大了再考慮跟 `topics/` 一樣做 `_templates/` 抽象）。

---

## 常見錯誤對照

| 症狀 | 原因 | 解法 |
|---|---|---|
| 同一主題被拆成兩個 profile | 影片重新下載後檔名沒保留原前綴 | 改名時對照 `audio_transcribe` 的 `profiles/REGISTRY.json` 確認正確前綴 |
| YouTube 描述出現奇怪括號 | handoff 樣板的 `<<>>`／`【】` 沒清乾淨 | 貼上前肉眼再檢查一次；根源已在 `audio_pipeline.ps1` v1.0.12 修過 |
| 播放清單建了卻是空的 | 建立時忘記先「新增影片」就按了建立 | 回播放清單編輯，補加影片 |
| 留言/描述裡的檔名變成超連結 | `.md`／`.io`／`.ai` 等副檔名剛好也是國家域名，平台自動誤判 | 講到檔名時避免直接寫出「字.副檔名」格式，或乾脆貼真正的完整網址 |
