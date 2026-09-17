# CLAUDE.md
Albert.Chou
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
- 一樣受本檔案「內容規範」的總規範約束（見下方）

## `_log/publish_log.md` 是發布歷史的唯一真實來源

- 每次新主題**發布完成**（YouTube 上架、social post 發出等動作），都要在 `_log/publish_log.md` 補一筆紀錄
- 要查「這個主題是什麼時候發的、發了哪些平台」，一律**先看這份 log**，不要用資料夾建立時間或其他線索推測
- 格式沿用既有表格欄位：日期／主題／動作／連結／備註

## 內容規範：Markdown 為主，允許程式碼，但公開曝險優先於方便

- 這個 repo 以 **Markdown 內容為主**，**也允許放程式碼**（原始碼、腳本、CI 設定等工程檔案）。主題若有對應的自動化／工具程式碼，可以直接放在該主題資料夾底下（例如 `topics/YYYY-MM-DD_主題slug/scripts/`），跟該主題的技術紀錄放在一起，方便查閱與維護
  - GitHub Actions workflow 檔案受平台限制，一律要放在 repo 根目錄的 `.github/workflows/`（無法放進子資料夾）；多個主題的 workflow 共存時，**檔名請加上主題 slug 前綴**避免混淆，並在 workflow 內用相對路徑指向對應主題資料夾下的程式碼
- **【安全規則，優先於上面的方便性考量】這個 repo 是公開（Public）的**。任何要放進來的程式碼、其執行產生的狀態檔（佇列 JSON、log 等），都**不能包含**：
  - API 金鑰、Token 等機密——一律走 GitHub Secrets，絕不寫死在程式碼或 commit 進 repo
  - 任何「本來就要保密到正式公開那一刻」的識別碼或內容（例如 YouTube **Unlisted 影片的 video_id**——這種連結不需要搜尋、只要有 ID 就能看，一旦寫進公開 repo 的 git 歷史，等於在正式發布前就先洩漏）
  - **若一個自動化的核心功能本質上需要「保密到公開那一刻」（例如排程延遲公開這類設計），該自動化的程式碼與狀態檔應該維持放在獨立的 Private repo**，content-hub 這邊只保留技術規劃紀錄、部署踩坑心得，兩邊用連結互相參照（範例：`topics/2026-09-16_youtube-auto-publish-scheduler/` 的技術紀錄放這裡，程式碼與含未公開 video_id 的佇列狀態放在獨立的 Private repo `yt-auto-publish`，不搬進來）
  - 沒有這類保密需求的一般自動化／工具（例如純資料彙整、不涉及未公開內容的排程任務），可以直接把程式碼放進 content-hub 對應主題資料夾
  - **判斷流程（規劃新自動化主題時一律先走這一步）**：先判斷是否涉及「未公開內容」。有 → 維持獨立 Private repo；沒有 → 可直接寫進 content-hub 對應主題資料夾。**判斷不出來、或有任何疑慮時，一律先預設當作「有保密需求」處理**（維持獨立 Private repo），並主動告知使用者、由使用者做最終決策——不要自行認定「應該沒關係」就直接放進公開的 content-hub
- **不放圖片／影片等二進位檔**（截圖、影片檔請放在其他地方，這裡只放連結）
- 例外：主題資料夾內的 `notebooklm_sources/` 可放 `.html` 文字來源文件（用途見上方「新增主題的標準流程」第 5 點）
- 目的：內容以 Markdown 為主維持可檢索性，程式碼跟著對應主題放一起方便維護追蹤；但「這個 repo 任何人都看得到」這件事永遠是第一考量，方便性不能凌駕於此

## 開發模式提醒

- **Cloud session 為主力**：跨裝置接續撰寫／編輯內容，是主要工作模式
- **Local session 僅做臨時小幅修改**：改完務必 `commit + push` 回 GitHub，不要留在本機未同步
