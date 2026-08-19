# CLAUDE.md

本檔案為 Claude Code 在此 repo 工作時的指引。

## 這個 repo 是什麼

`content-hub` 是 SIT 驗證與自動化實戰的**內容飛輪紀錄庫**：

```
技術工作 → GitHub 紀錄 → YouTube 影片 → NotebookLM
```

- **GitHub**：SEO 錨點，長期可被搜尋、引用的文字紀錄
- **YouTube**：示範教學層，把文字紀錄轉成可視化的操作教學
- **NotebookLM**：音訊再利用層，把既有內容轉成 podcast／音訊形式再擴散

三者環環相扣，同一份技術工作素材會依序流過這三層，而不是各自獨立產出。

## 新增主題的標準流程

1. **命名慣例**：`topics/YYYY-MM-DD_主題slug/`（日期為主題首次發布日）
2. **每個主題資料夾需包含三個檔案**：
   - `README.md`
   - `youtube_metadata.md`
   - `social_posts.md`
3. **一律從範本複製起手，不要憑空手寫**：
   - `README.md` ← `_templates/README_template.md`
   - `youtube_metadata.md` ← `_templates/youtube_metadata_template.md`
   - `social_posts.md` ← `_templates/social_posts_template.md`
   - 若需要腳本草稿，另有 `_templates/youtube_script_template.md`
   - 若是交接給 Cowork／Claude Code 開新任務，用 `_templates/cowork_task_prompts.md` 起手
4. **新主題建立後，記得回填**：
   - 根目錄 `README.md` 的「主題索引」表格新增一列
   - `_log/publish_log.md` 補一筆發布紀錄（見下）
5. **文字先行（尚未錄影／影片待錄）的主題，額外建立 `notebooklm_sources/`**：
   - 用途：放原始技術文件（`.md` / `.html`），作為繞過 Whisper 轉錄、直接手動餵給 NotebookLM 的標準文字進入點
   - 只放文字／HTML 來源，**不放音檔或影片檔**
   - 待該主題的 YouTube 影片實際錄製、逐字稿產出後，`notebooklm_sources/` 的角色可視情況保留（作為原始素材存檔）或由逐字稿取代

## `daily-tools/` — 每日工具挖掘候選池（獨立於 `topics/` 之外的頂層分類）

- **不是 `topics/` 的子項目**：`daily-tools/<YYYY-MM-DD>_<slug>/source.md` 跟 `topics/` 是平行的頂層分類，不套用上面「新增主題的標準流程」那一套規則
- **用途**：`daily-tool-digest` 自動化流程每日產出的候選工具介紹，作為 NotebookLM 音訊生成的原始素材
- **屬性：輕量候選池，不是正式主題**——不需要 `README.md`、`youtube_metadata.md`、`social_posts.md`，也不需要回填 `_log/publish_log.md`
- 一樣受本檔案「只放 Markdown」的總規範約束（見下方「內容規範」）

## `_log/publish_log.md` 是發布歷史的唯一真實來源

- 每次新主題**發布完成**（YouTube 上架、social post 發出等動作），都要在 `_log/publish_log.md` 補一筆紀錄
- 要查「這個主題是什麼時候發的、發了哪些平台」，一律**先看這份 log**，不要用資料夾建立時間或其他線索推測
- 格式沿用既有表格欄位：日期／主題／動作／連結／備註

## 內容規範：只放 Markdown，保持輕量

- 這個 repo **只放 Markdown 內容**
- **不放程式碼**（沒有原始碼、腳本、CI 設定等工程檔案）
  （例外：允許 `.github/workflows/` 僅用於儲存庫生命週期之 LINE 事件通知，嚴禁加入內容生成或爬蟲等工程邏輯）
- **不放圖片／影片等二進位檔**（截圖、影片檔請放在其他地方，這裡只放連結）
- 例外：主題資料夾內的 `notebooklm_sources/` 可放 `.html` 文字來源文件（非程式碼、非二進位檔，用途見上方「新增主題的標準流程」第 5 點）
- 目的是保持 repo 輕量、易於檢索與長期維護

## 開發模式提醒

- **Cloud session 為主力**：跨裝置接續撰寫／編輯內容，是主要工作模式
- **Local session 僅做臨時小幅修改**：改完務必 `commit + push` 回 GitHub，不要留在本機未同步
