# ChouAP Tunes：技術轉唱歌／音樂獨立子品牌線規劃

> 內容策略規劃：把「技術內容改編成歌曲」拉出來當一條獨立子品牌線，跟現有技術系列（`series/`）完全分開存放、分開發布

| 項目 | 內容 |
|---|---|
| 日期 | 2026-09-21 |
| 分類 | 自動化工具 |
| 狀態 | 草稿 |
| 子品牌名稱 | ChouAP Tunes |
| YouTube | [@ChouAPTunes](https://www.youtube.com/@ChouAPTunes)（獨立頻道，2026-10-03 由原本的「Biahal」頻道改名而來） |
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
- **實際執行狀況（2026-09-23）**：使用者已直接訂閱 **Pro Plan（月繳）**，跳過了原訂「先免費驗證、方向確定再升級」的順序——如實記錄實際發生的順序，不是照原計畫走。帳號畫面第一手確認方案內容：2,500 credits／月、**20 次下載／月**、下次扣款日 2026-10-23，跟先前查到的公開資料一致。另外發現 **WAV 高音質下載是 Pro 專屬功能**（免費版沒有），正式發布用的檔案建議選 WAV 格式而非預設格式

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
- **修正（2026-09-29）：歌詞不是使用者親自寫的，作詞也是 AI 生成**——使用者的原創部分是技術內容本身（已發布的教學 YouTube 影片／SOP），**歌詞是透過使用者的教學 YouTube 工作流程，用 AI 把技術內容改編、轉譯成歌詞**；作曲／編曲／人聲演唱則由 Suno AI 生成。正確的情境描述是：**技術內容原創，作詞、作曲、演唱全部經 AI 生成**，且不模仿特定真人歌手聲音（voice cloning）
- 因為歌詞本身也非人工手寫，比原本評估的「使用者自己寫詞」更接近全 AI 生成，**建議上傳時一律把 YouTube Studio「Altered or synthetic content」欄位勾是**，採取比較保守保險的做法
- **真正的紅線仍是 voice cloning（複製／模仿特定真人歌手的聲音）**，不是揭露問題本身——只要不做這件事，AI 作詞＋作曲＋演唱這件事本身沒有版權疑慮

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

## 執行狀況（2026-10-08 更新）

- **發布進度**：第 1 到 6 首已公開（發布日期依使用者 2026-10-05 的 Studio 截圖：第 1 首 2026-10-01、第 2 首 10-02、第 3 首 10-03、第 4 首 10-04；第 5 首《定格》10-06、第 6 首《沒睡著的那一顆》10-08 由排程自動公開；連結見 `_log/publish_log.md`）。第 1 首已由使用者在 Studio 正式改名為《不變的信仰》（原名《接地永遠在通電之前》）。**第 7、8 首已完成製作、經人工核對字幕與開場留言並核准**，依序每 48 小時公開 1 首，預計台灣時間 10/10、10/12 凌晨（GitHub 的排程常晚幾個小時，第 5 首就是 08:13 才公開）；各首公開後，歌名與連結才回填到發布紀錄（公開前不寫歌名，維持「保密到正式公開那一刻」的規範）
- **影片製作**：歌曲以 `chouap-tunes`（Private）裡的工具自動組裝成 MP4（依時間軸設定把背景圖、動態片段、母帶與字幕合成，本機有 NVIDIA 顯卡時可用 GPU 加速）；字幕走本機 Demucs 人聲分離加 Whisper 辨識，再用校對規則與漏句檢查把關
- **發布自動化**（實作在私有 repo `yt-auto-publish`，跟技術頻道共用同一個 repo，但路徑、憑證、排程完全各自獨立）：每天台灣時間 02:30 觸發；**兩階段把關**——先上傳到私人驗證播放清單，人工在 Studio 核對字幕與開場留言後核准，下一次排程才自動轉公開、移入「ChouAP Tunes｜全部歌曲」並發 LINE 通知；**每 48 小時最多公開 1 首**（距離上一次公開未滿 36 小時就不公開），所以就算同時準備好多首，也不會一次發光
- **互動分析日報**（第三條獨立經路，台灣時間每天 07:00）：抓「全部歌曲」播放清單裡每支影片的觀看、讚、留言數，跟前一天比對，再由 Claude 寫分析與建議，LINE 發摘要、Gmail 寄全文。3 個 Secret 已於 2026-10-05 設定，手動執行驗證成功（LINE 摘要與 HTML 排版的 Gmail 完整報告都已收到），之後每天由排程自動執行。觀眾留言屬於不可信的輸入，已對這一步加上防護，避免有人用留言指示模型改動 repo 的腳本
- **公開紀錄的連結規則**（2026-10-05）：指向 Private repo（`chouap-tunes`、`yt-auto-publish`）的超連結已從公開紀錄全部移除，只寫名稱與路徑，規則寫在 `CLAUDE.md`

## 待決策事項

- [x] 子品牌正式名稱 → **ChouAP Tunes**
- [x] 新 repo Public 或 Private → **先 Private 起手**，方向定調、命名穩定後再切 Public
- [x] 沿用既有 YouTube 頻道另開播放清單，還是開全新頻道 → **獨立頻道**：2026-10-03 把原本的「Biahal」頻道改名為 **ChouAP Tunes**（`@ChouAPTunes`），跟技術頻道「Albert Biahal」（`@AlbertBiahal`）完全分開，兩邊可互相導流；2026-10-05 使用者的頻道頁截圖確認名稱與帳號代碼正確（目前 4 部影片）
- [x] AI 作曲工具選型 → **Suno**（Udio 已於 2025-10-30 起停用下載匯出功能，無法用於這條產線，見上方「重大更新」）；**Pro Plan 已訂閱（月繳，2026-09-23）**，商用授權隨方案生效；YouTube Content ID 風險仍待實際上架後觀察
- [x] 挑選第一支試作內容做 pilot → **DSOX4024G 三步驟電壓量測**（已發布技術影片改編）；經多輪歌詞／Style 調整（洗腦上口版 → 放慢節奏品味人生版 → 激昂副歌＋低沉明亮＋押韻版），**第一版母帶已產出且效果達標**（2026-09-23）。完整歌詞、Style 參數、母帶位置記錄於 `chouap-tunes/pilots/001-dsox4024g.md`（`chouap-tunes` 已建立 pilots 總索引，之後多曲風擴充與 Gemini CLI 讀取都以這張索引表為準）

## 行動項（若有後續待辦）

- [x] ~~使用者拍板子品牌名稱與新 repo 的 Public／Private 屬性後，建立新獨立 repo~~ → 名稱與屬性已拍板，**嘗試建立 `chouap-tunes`（Private）失敗**：這個 content-hub session 掛的 GitHub App 只被授權存取 `content-hub` 這一個 repo，沒有帳號層級的「建立新 repo」權限（API 回傳 `403 Resource not accessible by integration`）
- [x] ~~需要使用者手動建立 repo~~ → 使用者已手動建立完成：`chouap-tunes`（Private，空 repo，未初始化 README／.gitignore／license）
- [x] ~~若要讓 Claude 之後直接操作 `chouap-tunes`，需要使用者另外授權 Claude GitHub App~~ → 使用者已完成授權，Claude 已 clone 並推送初始 `README.md`（說明定位與現況，連回本篇規劃紀錄）
- [x] ~~使用者接下來前往 Suno 註冊帳號，評估付費方案~~ → 已完成，並直接訂閱 Pro Plan（見上方「執行狀況」）
- [x] ~~選定 pilot 內容並產出第一首試作歌曲，驗證效果~~ → 已完成，「接地永遠在通電之前」達標，母帶已下載（WAV），存放於 `G:\我的雲端硬碟\YouTube工作流\音樂題材\母帶\DSOX4024G_接地永遠在通電之前\`（本機路徑，Claude 無法存取，純記錄位置）
- [x] ~~未來工作流規格書位置~~ → 已記錄於 `chouap-tunes` README 與本篇「延伸資源」
- [x] ~~決定「接地永遠在通電之前」是否作為 ChouAP Tunes 第一支正式發布內容；若要發布，需另外剪輯 MP4~~ → 已發布（2026-10-01，Studio 顯示的發布日期）；MP4 由本機 `build_video.py` 依時間軸設定自動組裝，不再手動剪輯
- [x] ~~再決定是否要把這套「達標公式」套用到下一支技術內容~~ → 已啟動第二支 pilot（見下）
- [x] ~~第二支 pilot（002）技術內容選定~~ → 確定為 [ChouAP.Cloud 上雲 SOP](../2026-08-12_chouap-cloud-migration-sop/README.md)（已發布），曲風 Pop Rock／沙啞渾厚磁性煙嗓男聲，主題扣緊「自由、不依賴單一台電腦」的敘事；歌詞草稿已產出，記錄於 `chouap-tunes/pilots/002-pop-rock-smoky-voice.md`，待使用者於 Suno 測試效果
- [x] ~~第二支 pilot 測試結果回報後，補上母帶位置，更新索引狀態~~ → 已完成，歌名定案「**雲端浪人**」，母帶已產出且效果達標（2026-09-24），存放於 `G:\我的雲端硬碟\YouTube工作流\音樂題材\母帶\上雲 SOP 不依賴單一台電腦_雲端浪人\`（本機路徑，Claude 無法存取）
- [x] ~~決定「雲端浪人」是否作為正式發布內容~~ → 已公開（2026-10-02 發布，使用者 Studio 截圖確認）；第 3 首「沉默答案」2026-10-03 也已公開
- [x] ~~第三支 pilot（003）曲風與技術內容選定~~ → 來源技術內容確定為 [YouTube 影片排程自動發布系統](../2026-09-16_youtube-auto-publish-scheduler/README.md)（草稿階段），把系統「靜默背叛」的踩坑細節（OAuth 7 天靜默失效、配額用盡靜默跳過、60 天未動被停用）包成感情破裂敘事
- [x] ~~第三支 pilot 測試與風格調整~~ → 歌名定案「**沉默答案**」，人聲經 6 輪調整（磁性煙嗓女聲→男聲→調 BPM→拿掉煙嗓改渾厚→男聲加 mature/sensual→換回女聲再加強），最終定案為**深度成熟、感性、煙嗓的磁性女聲（alto 音域）**，母帶已產出且效果達標（2026-09-24），存放於 `G:\我的雲端硬碟\YouTube工作流\音樂題材\母帶\OAuth七天靜默失效配額用盡靜默跳過60天沒動靜就被停用_沉默答案\`（本機路徑，Claude 無法存取）；完整調整歷程記錄於 `chouap-tunes/pilots/003-emo-smoky-female-voice.md`
- [x] ~~字幕產線方案確定~~ → CapCut 自動歌詞額度有限、YouTube 自動字幕因唱歌＋中英混雜技術詞辨識失敗，最終改用**本機 Demucs 人聲分離 + Whisper 辨識**，公司（CPU）與家用（RTX GPU）電腦分工建置成功，免費無限次；用這套管線校對 001 時發現**實際演唱結構跟原始歌詞不同**（整段 Bridge 被跳過、改成多唱一次副歌），剪輯排列建議已修正。詳細安裝步驟、指令與踩坑記錄於 `chouap-tunes/pilots/001-dsox4024g.md`「字幕產線」章節
- [x] ~~第 4 首（004）走完兩階段發布流程並公開上架~~ → 2026-10-04 完成，第一支走完「上傳到私人驗證播放清單 → 人工核對 → 核准 → 排程自動公開」的歌（紀錄見 `_log/publish_log.md`）
- [x] ~~發布自動化：兩階段把關、每 48 小時最多公開 1 首~~ → 2026-10-03 至 04 完成（實作在私有 repo `yt-auto-publish`，細節見本篇「執行狀況」）
- [x] ~~第 5 到 8 首完成製作並核准，排入公開佇列~~ → 2026-10-05 完成，預計 10/6、10/8、10/10、10/12 凌晨依序公開；公開後回填歌名與連結到 `_log/publish_log.md`
- [x] ~~互動分析日報（第三條獨立經路）程式碼就位~~ → 2026-10-05 完成
- [x] ~~設定互動分析日報需要的 3 個 GitHub Secret（Claude Code 認證 Token、收發報告的 Gmail 地址、該帳號的應用程式密碼），再到 Actions 手動執行一次驗證~~ → 2026-10-05 完成
- [ ] 第 5 到 8 首每首公開後：到 YouTube Studio 釘選開場留言（API 無法釘選），並回填公開連結到發布紀錄（第 5 首 2026-10-06、第 6 首 2026-10-08 已完成）
- [x] ~~第 1 到 3 首的公開日期補進發布紀錄~~ → 2026-10-05 已補（10-01、10-02、10-03，依使用者的 Studio 截圖）
- [x] ~~第 2、3 首的公開連結補進發布紀錄~~ → 2026-10-05 完成（第 1 到 4 首的公開連結都已在 `_log/publish_log.md`）

## 延伸資源

- 相關文章／前作（技術系列策展邏輯參考，注意此篇的子品牌線刻意跟這份文件描述的三系列分開）：[`series/README.md`](../../series/README.md)
- 分工模式參考（獨立 repo＋content-hub 規劃紀錄互相參照，但拉出原因不同——見上方「關鍵發現」）：[YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃](../2026-09-16_youtube-auto-publish-scheduler/README.md)
- 相關 repo：`chouap-tunes`（Private，歌曲設計文件與影片製作工具，Claude 已取得存取權）、`yt-auto-publish`（Private，ChouAP Tunes 的發布自動化與互動分析日報，跟技術頻道共用 repo、各自獨立路徑）
- AI 作曲工具：[Suno](https://suno.com)（目前選定使用的工具，見上方「AI 作曲工具費用」與「重大更新」）
- 未來工作流交接位置（本機路徑，Claude 無法存取）：`G:\我的雲端硬碟\產品上架(Product Release Pipeline)\設計規範\Agy_Antigravity\曲音聲專案`，交給 Gemini CLI（Agy_Antigravity）執行這條產線的工作流

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
