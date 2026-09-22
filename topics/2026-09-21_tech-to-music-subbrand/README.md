# ChouAP Tunes：技術轉唱歌／音樂獨立子品牌線規劃

> 內容策略規劃：把「技術內容改編成歌曲」拉出來當一條獨立子品牌線，跟現有技術系列（`series/`）完全分開存放、分開發布

| 項目 | 內容 |
|---|---|
| 日期 | 2026-09-21 |
| 分類 | 自動化工具 |
| 狀態 | 草稿 |
| 子品牌名稱 | ChouAP Tunes |
| YouTube | |
| NotebookLM | |

## 背景

content-hub 原本的內容飛輪是單向的「技術工作 → GitHub 紀錄 → YouTube 影片 → NotebookLM」，`series/README.md` 另外記錄了三個技術系列（DSOX4024G 示波器實戰、AI機槽成長歷程、Claude Code 雲端開發），共用同一套 `audio_transcribe` repo 的錄音轉錄／NotebookLM／字幕回爐管線。

討論中提出一個新方向：把技術內容「透過其他方式（唱歌、音樂）」再導回 YouTube，例如用 AI 作曲工具把技術 SOP 改編成歌詞、產生歌曲當作 Shorts 或另類科普素材。經過討論確認兩個關鍵決策：

1. **不能跟現有技術系列放在一起**：技術系列已經是成熟穩定的工作流（`audio_transcribe` 靠檔名前綴比對 profile），音樂改編是完全不同的創作動作，硬塞進去會打亂 profile 判斷邏輯，兩者的品牌調性（專業教學 vs 洗腦／娛樂向）也會互相稀釋
2. **整條線要拉出來，開全新獨立 repo**：不只是資料夾層級的區隔，而是連工程層（未來若做自動化腳本、素材管理）都要跟 content-hub／`audio_transcribe`／`yt-auto-publish` 完全分開，比照 `yt-auto-publish` 的分工模式——content-hub 這邊只留規劃紀錄與連結互相參照，實際程式碼與素材放獨立 repo

子品牌正式名稱已拍板為 **ChouAP Tunes**，延伸既有 ChouAP.Cloud 品牌識別。新 repo 決定**先 Private 起手，等方向定調、命名與內容都穩定後再切成 Public**——這跟 `yt-auto-publish` 需要永遠保持 Private 不同，這裡只是「還沒準備好見人」的過渡狀態，GitHub 允許事後把 Private 轉 Public 且 commit 歷史完整保留，不會有轉換損失。

## 做了什麼（規劃中的架構）

四層拆分，比照 `series/README.md` 的策展邏輯，但整條線物理上獨立：

1. **定位層**：獨立子品牌 **ChouAP Tunes**，暫不掛在既有 YouTube 頻道的技術系列播放清單下，視覺識別（縮圖風格）待定
2. **內容層**：挑選已發布的技術教學／SOP 內容，改編歌詞，用 AI 作曲工具產生歌曲（工具選型見下方「前置研究」，目前傾向 Suno，理由見下）
3. **通路層**：獨立 YouTube 播放清單，是否需要開全新頻道（而非既有頻道底下的播放清單）待決定
4. **工程層**：**全新獨立 GitHub repo（`chouap-tunes`，Private 起手）**，未來若有自動化腳本（例如批次產歌詞草稿、素材版控）程式碼放這裡；content-hub 只放這篇規劃紀錄，不放歌曲音檔／影片（依 `CLAUDE.md` 二進位檔規範）

## 關鍵發現／重點

- **為什麼要物理隔離，不只是資料夾隔離**：`series/README.md` 記載的 STEP 5.5 提到，`audio_transcribe` 的 profile 判斷是靠錄音檔名前綴比對，音樂改編內容如果混進同一套素材管理／檔名規則，容易被誤判成既有系列的新一集，反而製造混亂
- **記憶點 vs 品牌調性是核心 trade-off**：歌曲化內容比純語音 Podcast（NotebookLM 音訊）更有記憶點、更適合剪 Shorts 病毒擴散，但跟目前頻道「SOP／自動化實戰」的專業調性反差大，這也是決定「拉出獨立子品牌」而非「掛在現有頻道下」的主因
- **跟 `yt-auto-publish` 的分工模式不同之處**：`yt-auto-publish` 拉出獨立 repo 是因為「保密到公開那一刻」的安全考量（未公開 video_id 不能進公開 repo）；這次拉出獨立 repo **不是保密需求，是品牌區隔需求**——兩者都用「獨立 repo＋content-hub 留規劃紀錄互相參照」的分工外殼，但背後原因不同，未來若有人回頭看這篇要留意別混為一談

## 前置研究：工具費用與法遵確認（2026-09-22 查詢，僅供參考，動手前需回官方頁面核對最新數字）

### AI 作曲工具費用（Suno／Udio）

- **Suno**：免費版每日 50 credits、有浮水印、僅供個人非商業用途；付費方案才有商用授權——Pro US$10/月（2,500 credits／月，最多 500 首）、Premier US$30/月（10,000 credits／月，最多 2,000 首）
- **Udio**：免費版每月 100 credits（每 24 小時上限約 10 credits）；付費方案——Standard US$10/月（2,400 credits）、Pro US$30/月（6,000 credits）
- **結論**：這條線的內容要拿去 YouTube 上架、可能有廣告收益，屬於商業使用，**免費版不夠用，需要至少 Pro／Standard 等級付費方案**，抓每月 NT$300–1,000 左右的營運成本

### ⚠️ 重大更新（2026-09-22 查詢）：Udio 目前無法下載匯出，實務上不能用於這條產線

- Udio 於 2025-10-30 與 Universal Music Group 達成版權訴訟和解後，**停用了所有下載／匯出功能**，只在 2025-11-03～11-05 開放 48 小時讓使用者搶救舊作品，之後就沒再恢復
- Udio 目前是「**串流限定的封閉花園（walled garden）**」：可以在平台內生成、播放音樂，但**無法匯出檔案上傳到 YouTube 或任何外部平台**——等於不管付不付費，Udio 現在都做不到「產出一首歌、下載下來、剪進 YouTube 影片」這個 ChouAP Tunes 需要的最基本動作
- Udio 官方說 2026 年會推出跟 UMG 合作的新一代授權平台、屆時「希望」能重新開放下載，但沒有確定時程
- **結論：目前 AI 作曲工具選型直接排除 Udio，實質上只剩 Suno 可用**（付費方案含下載與商用授權），待決策事項下方同步更新

### 亮點功能比較（僅供對照參考，Udio 已不能匯出，實際只能選 Suno）

**Suno**：
- 人聲最自然、完整度最高，公認是目前 AI 作曲工具中人聲表現最好的
- **Suno Studio**：瀏覽器內的多軌編輯環境，可拉時間軸編排、控制 BPM、拆分音軌（stem）、匯出 MIDI／音訊，功能接近簡易 DAW
- v5.5 版本另有 voice cloning 功能（**注意**：這正是前面提到 YouTube 揭露規範的紅線，若之後真的用到需另外評估）
- 付費方案含商用授權與下載，是目前唯一能真正「產出可用檔案」的選項

**Udio**（僅記錄其技術特色，目前無法實際採用）：
- 人聲真實度與細緻編輯控制曾被認為業界領先，尤其器樂／爵士／古典／環境音樂的音質評價高（48kHz 立體聲輸出）
- **Inpainting（分段重新生成）**：可針對歌曲局部段落單獨重新生成，不用整首重錄，曾是它的獨門功能
- 社群功能較強，可瀏覽其他使用者作品、remix、探索風格
- 但如上所述，**這些功能目前都因為無法匯出檔案而用不上**，只能等 2026 年 UMG 授權新平台上線後看是否恢復下載

### YouTube AI 揭露規範

- 判斷要不要揭露的關鍵是「**內容是否看起來像真實、會不會讓觀眾誤以為是真人真事**」，不是「概念是不是自己原創設計」
- 使用者情境（自己寫技術內容／歌詞，AI 只負責生成人聲演唱，不模仿特定真人歌手聲音）通常落在**不強制揭露**的範疇；仍建議上傳時把 YouTube Studio「Altered or synthetic content」欄位勾是，較保險
- **真正的紅線是 voice cloning（複製／模仿特定真人歌手的聲音）**，不是揭露問題本身——只要不做這件事，自己寫詞、AI 生成演唱這件事本身沒有版權疑慮

### 商標申請（若之後要保護子品牌名稱）

- 主管機關：經濟部智慧財產局（TIPO），官網「商標檢索系統」可先免費查詢是否已有相似名稱
- 費用（2026 現行規費）：申請規費 NT$3,000／類別（電子送件＋完全使用系統內建商品／服務參考名稱，最低可折抵到 NT$2,400／類）＋核准後註冊公告費 NT$2,500／件，一個類別粗抓約 **NT$5,000 左右**；若需加速審查另加 NT$6,000
- 流程：檢索確認無衝突 → 準備商標圖樣與指定類別 → 電子申請（需自然人憑證等數位憑證）→ 官方審查 6–9 個月（有補正意見可能拖到 12 個月以上）→ 核准後繳費領證
- 可自行送件（省錢，需花時間研究流程），也可找商標代理人代辦（行情約 NT$5,000–15,000，看事務所）

### 參考來源

- [Suno Pricing 2026: Cost & Competitor Comparison](https://checkthat.ai/brands/suno/pricing)
- [Suno AI Pricing 2026: Free, Pro & Premier Plans Explained](https://sunowatermark.com/blog/suno-ai-pricing-2026/)
- [Udio pricing 2026: A complete breakdown of plans & credits](https://www.eesel.ai/blog/udio-pricing)
- [Udio Pricing 2026: Free vs Standard vs Pro](https://margabagus.com/udio-pricing-plans/)
- [How we're helping creators disclose altered or synthetic content - YouTube Blog](https://blog.youtube/news-and-events/disclosing-ai-generated-content/)
- [Disclosing use of altered or synthetic content - YouTube Help](https://support.google.com/youtube/answer/14328491?hl=en)
- [AI Music on YouTube: Allowed With Disclosure Rules](https://dynamoi.com/learn/ai-music-distribution/is-ai-music-allowed-on-youtube)
- [商標申請費用 2026 全解析](https://www.nss.com.tw/trademark-application-taiwan)
- [智慧財產局商標主題網－商標規費清單](https://www.tipo.gov.tw/tw/trademarks/589.html)
- [商標申請怎麼做？2026 最新商標註冊流程、費用、時間](https://liheng-law.com/trademark-registration-application-guide/)
- [Udio Halts AI Song Downloads After Copyright Settlement with UMG, Warner](https://www.webpronews.com/udio-halts-ai-song-downloads-after-copyright-settlement-with-umg-warner/)
- [Changes associated with the Universal Music Group ("UMG") partnership - Udio Help Center](https://help.udio.com/en/articles/12683565-changes-associated-with-the-universal-music-group-umg-partnership)
- [Suno vs Udio 2026: The AI Music Generation Showdown](https://neuronad.com/suno-vs-udio/)

## 待決策事項

- [x] 子品牌正式名稱 → **ChouAP Tunes**
- [x] 新 repo Public 或 Private → **先 Private 起手**，方向定調、命名穩定後再切 Public
- [ ] 沿用既有 YouTube 頻道另開播放清單，還是開全新頻道
- [x] AI 作曲工具選型 → **Suno**（Udio 已於 2025-10-30 起停用下載匯出功能，無法用於這條產線，見上方「重大更新」）；仍需確認 Suno 付費方案的商用授權細節與 YouTube Content ID 風險
- [ ] 挑選第一支試作內容做 pilot（建議從已發布的技術影片挑一支改編，成本最低、有基礎素材可比對效果）

## 行動項（若有後續待辦）

- [x] ~~使用者拍板子品牌名稱與新 repo 的 Public／Private 屬性後，建立新獨立 repo~~ → 名稱與屬性已拍板，**嘗試建立 `chouap-tunes`（Private）失敗**：這個 content-hub session 掛的 GitHub App 只被授權存取 `content-hub` 這一個 repo，沒有帳號層級的「建立新 repo」權限（API 回傳 `403 Resource not accessible by integration`）
- [ ] **需要使用者手動建立 repo**：登入 GitHub → 右上角 `+` → `New repository` → repo name 填 `chouap-tunes` → 選 **Private** → 建立；建立後把連結回覆給 Claude，或用 `add_repo` 工具把新 repo 加進 session 範圍，即可繼續回填本篇「延伸資源」連結、視需要協助初始化 repo 內容（例如仿照 `yt-auto-publish` 加一份 CLAUDE.md）
- [ ] 新 repo 建立後，回填本篇「延伸資源」的 repo 連結
- [ ] 選定 pilot 內容並產出第一首試作歌曲，驗證效果後再決定是否往自動化 pipeline 推進

## 延伸資源

- 相關文章／前作（技術系列策展邏輯參考，注意此篇的子品牌線刻意跟這份文件描述的三系列分開）：[`series/README.md`](../../series/README.md)
- 分工模式參考（獨立 repo＋content-hub 規劃紀錄互相參照，但拉出原因不同——見上方「關鍵發現」）：[YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃](../2026-09-16_youtube-auto-publish-scheduler/README.md)
- 相關 repo：`chouap-tunes`（Private，待使用者手動建立，見上方「行動項」）

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
