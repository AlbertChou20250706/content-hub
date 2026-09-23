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
- **執行策略（拍板）**：先用 **免費版驗證**——每天 50 credits、一次生成約扣 5–10 credits，等於一天可測 5–10 次，足夠驗證「技術內容改編歌詞、AI 演唱效果」這個階段；等驗證方向可行、確定要正式發布時再升級 **Pro（US$10/月）**。**注意：免費版產出檔案有浮水印、授權僅供個人非商業用途，不能直接拿去 YouTube 正式發布**——正式發布前必須用升級後的 Pro 帳號重新生成一次當最終檔案，不能沿用免費驗證期做的版本
- **⚠️ 重大更新（2026-09-23 查詢）：Suno 自 2026-09-03 起改下載政策，改成配額制且回溯適用**：**免費版是終身總共 7 次下載**（不是每天／每月重算），**Pro 每月 20 次、Premier 每月 60 次**；免費版下載仍僅供個人非商業用途。這個限制只影響「下載存檔」，在網頁內生成／播放試聽不受影響，一樣吃當天的 credits 額度。**驗證階段策略調整**：多在網頁裡生成、試聽篩選，篩到真的想留底比對的版本才下載，把 7 次終身額度留給值得留存的版本，不要每個草稿都下載；正式發布用的檔案仍必須等升級 Pro 後重新生成＋下載
- **踩坑紀錄（2026-09-22）**：註冊時用 **Google SSO 登入持續失敗**（一般視窗與無痕視窗都試過，畫面卡在 `auth/verify` 驗證步驟後跳「Log in failed」），改用 **Microsoft SSO 登入一次成功**；Suno 僅支援 SSO（Apple／Discord／Facebook／Google／Microsoft），沒有 email＋密碼登入，判斷是 Google 授權那端當下的暫時性問題，不是 Suno 或本機瀏覽器的問題——之後若又遇到 Google 登入失敗，直接換一個 SSO 供應商試即可，不用花時間查瀏覽器設定

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
- [Suno | An update to our downloads policy and Terms of Service](https://suno.com/blog/suno-updates-tos)
- [How many downloads come with my subscription? - Suno Help Center](https://help.suno.com/en/articles/13926209)
- [AI Music Company Suno Unveils Download Caps for Free, Paid Tiers - Variety](https://variety.com/2026/music/news/suno-unveils-download-caps-for-free-paid-tiers-generator-1236831589/)

## 待決策事項

- [x] 子品牌正式名稱 → **ChouAP Tunes**
- [x] 新 repo Public 或 Private → **先 Private 起手**，方向定調、命名穩定後再切 Public
- [ ] 沿用既有 YouTube 頻道另開播放清單，還是開全新頻道
- [x] AI 作曲工具選型 → **Suno**（Udio 已於 2025-10-30 起停用下載匯出功能，無法用於這條產線，見上方「重大更新」）；仍需確認 Suno 付費方案的商用授權細節與 YouTube Content ID 風險
- [ ] 挑選第一支試作內容做 pilot（建議從已發布的技術影片挑一支改編，成本最低、有基礎素材可比對效果）

## 行動項（若有後續待辦）

- [x] ~~使用者拍板子品牌名稱與新 repo 的 Public／Private 屬性後，建立新獨立 repo~~ → 名稱與屬性已拍板，**嘗試建立 `chouap-tunes`（Private）失敗**：這個 content-hub session 掛的 GitHub App 只被授權存取 `content-hub` 這一個 repo，沒有帳號層級的「建立新 repo」權限（API 回傳 `403 Resource not accessible by integration`）
- [x] ~~需要使用者手動建立 repo~~ → 使用者已手動建立完成：[`chouap-tunes`](https://github.com/AlbertChou20250706/chouap-tunes)（Private，空 repo，未初始化 README／.gitignore／license）
- [x] ~~若要讓 Claude 之後直接操作 `chouap-tunes`，需要使用者另外授權 Claude GitHub App~~ → 使用者已完成授權，Claude 已 clone 並推送初始 `README.md`（說明定位與現況，連回本篇規劃紀錄）
- [ ] 使用者接下來前往 [Suno](https://suno.com) 註冊帳號，評估付費方案（商用授權與下載需求，見上方「AI 作曲工具費用」）
- [ ] 選定 pilot 內容並產出第一首試作歌曲，驗證效果後再決定是否往自動化 pipeline 推進
- [ ] 未來工作流規格書會寫在使用者本機 Google 雲端硬碟路徑（見下方「延伸資源」），交給 Gemini CLI（Agy_Antigravity）執行；Claude 無法存取該路徑，純記錄位置供之後對照

## 延伸資源

- 相關文章／前作（技術系列策展邏輯參考，注意此篇的子品牌線刻意跟這份文件描述的三系列分開）：[`series/README.md`](../../series/README.md)
- 分工模式參考（獨立 repo＋content-hub 規劃紀錄互相參照，但拉出原因不同——見上方「關鍵發現」）：[YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃](../2026-09-16_youtube-auto-publish-scheduler/README.md)
- 相關 repo：[`chouap-tunes`](https://github.com/AlbertChou20250706/chouap-tunes)（Private，已建立並初始化 README，Claude 已取得存取權）
- AI 作曲工具：[Suno](https://suno.com)（目前選定使用的工具，見上方「AI 作曲工具費用」與「重大更新」）
- 未來工作流交接位置（本機路徑，Claude 無法存取）：`G:\我的雲端硬碟\產品上架(Product Release Pipeline)\設計規範\Agy_Antigravity\曲音聲專案`，交給 Gemini CLI（Agy_Antigravity）執行這條產線的工作流

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
