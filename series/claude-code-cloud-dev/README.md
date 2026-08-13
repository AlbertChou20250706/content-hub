# Claude Code 雲端開發實戰系列

> content-hub 系列｜對應 audio_transcribe Profile：`aidev`
> YouTube 播放清單：<補上播放清單網址>

## 系列定位

記錄如何用 Claude Code 打造雲端優先的開發工作流——從專案搬遷、CLAUDE.md 規範設計，到自動化內容生產管線本身的實戰過程。這個系列的內容特色是「拿自己實際踩過的坑當素材」，不是預先設計好的教學腳本，因此每一集的產出，本質上也是這套飛輪自我運作的證明。

## 目前集數

| # | 標題 | 發布日期 | 對應 topic | YouTube |
|---|---|---|---|---|
| 1 | <實際選定的候選標題> | 2026-08-13 | `topics/2026-08-12_chouap-cloud-migration-sop/` | <影片連結> |

## 這個系列的素材來源比較特別

跟其他系列不同，這個系列的內容天生就是「文字先行」——技術工作本身（腳本開發、SOP 設計）已經產出完整的 Markdown 文件，不需要另外錄音講一遍。素材放在對應 topic 的 `notebooklm_sources/` 資料夾，直接餵給 NotebookLM 生成影片口白，繞過 Whisper 逐字稿這一步。

## 內容方向（規劃中，非承諾）

- Cloud vs Local session 的選擇與實際踩坑經驗
- Profile 自我繁殖機制的長期實戰記錄
- 其他 Claude Code / AI 開發工作流主題，視後續實際遇到的問題而定

## 相關檔案位置

| 內容 | 位置 |
|---|---|
| 這個系列的 Profile 設定 | `audio_transcribe` repo → `profiles/aidev.json` |
| 專屬字典 | `audio_transcribe` repo → `glossary/aidev.md` |
| 素材來源（文字先行主題） | `content-hub` repo → `topics/<date>_<slug>/notebooklm_sources/` |
| 一般錄音主題的素材 | `audio_transcribe` repo → `recordings/`（僅本機/Drive，不進 Git） |
