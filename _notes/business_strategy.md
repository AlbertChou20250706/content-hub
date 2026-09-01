# 經營策略筆記

> 這份檔案放經營規劃、流程草案這類**內部筆記**，跟 `topics/`／`daily-tools/` 底下公開發布的內容分開。
> 之後說「去 content-hub 找」，指的就是來 `_notes/` 這裡翻。

## 內容變現流程（草案，之後可調整）

1. **選材**：找尋合適、有價值的工具
2. **GitHub 紀錄**：上傳／整理進 GitHub（SEO 錨點，這部分就是目前 `content-hub` + `daily-tool-digest` 在做的事）
3. **內容轉製**：設計轉錄／腳本，把技術素材轉成可發布的內容
4. **YouTube 上架**
5. **作品本身也上 GitHub**：把實際的工具／程式碼作品公開 release
6. **防呆機制**：分享歸分享，但要設計防止他人直接複製貼上整包程式碼的機制（保護作品）
7. **流量變現**：靠 YouTube + GitHub 兩邊的流量／曝光累積
8. **販售作品**（之後再討論，還沒定案）

## 第 5～8 步實現方案（草案）

### 原則

- 作品要在 GitHub 上「有貢獻」，同時有價值的程式碼不能被整包複製流出
- 販售採**小額買斷制**：一次性付費取得使用權，不含後續更新／維護保證，但買家可在 Issue 提問
- 初期用**多 repo 分工**，公開與私有分開管理

### 1. GitHub 貢獻可見度

即使核心程式碼放私有 repo，README、文件更新、release note、回覆 issue 都算貢獻活動，一樣能維持 contribution graph 綠格子。不必為了保護程式碼犧牲曝光度，兩者不衝突。

### 2. 買斷制＋無後續服務但可提問

不做 license key 系統，改用：**私有 repo ＋ 付款後加對方為 collaborator／read 權限**。條款寫清楚：

- 一次性買斷＝取得原始碼使用權
- 不含更新維護保證（不承諾 SLA）
- 但買家可在 Issue 提問，屬於社群式支援而非有償服務

### 3. 多 repo 分工

延續現有帳號慣例（PXE、SPD_Flash、PTU_CPU_Verify 各自一個 repo），訂成規範：

- 一個工具＝一個「公開展示 repo」（放 README＋編譯好的 release，如 PyInstaller 打包的 .exe）
- 對應一個「私有原始碼 repo」（放完整 `.py` 原始碼）
- 兩邊用同名＋ `-src` 後綴之類的命名規則對應，方便管理

### 4. 網站需求

- **初期不需要**獨立網站，先用既有 GitHub Pages 作品集（Albert-Pyahaw）撐住展示＋導流角色
- 等真的要做「付款＋授權管理」，優先接現成的 Gumroad／LemonSqueezy／Stripe Payment Link，不必自建整站
- 等規模夠大、需要自己的品牌落地頁時，再考慮自建

## 執行順序建議

不要一次把所有 repo 都拆開，先挑一個小工具（如 `OT_Calc_Launcher`）當試點：走一次「公開展示 repo ＋ 私有原始碼 repo ＋ 買斷加 collaborator」完整流程，跑通後再套用到其他 repo。

## 每日小工具素材（本機路徑）

- 素材來源：本機／Google 雲端硬碟 `G:\我的雲端硬碟\產品上架(Product Release Pipeline)\Scrip_application`，收集程式碼量較少的小工具，比照 `daily-tools/` 現有作法每天固定時間（構想 05:40）發一篇
- **限制**：Claude Code cloud session 存取不到本機 `G:\` 路徑，且依 `CLAUDE.md` 規範，content-hub 這個 repo 本身**不能**放抓取／生成邏輯的自動化（`.github/workflows/` 僅允許 LINE 通知用途）
- 這件事的正確歸屬是**擴充 `daily-tool-digest`**（獨立的自動化專案）的來源清單，讓它去讀 `Scrip_application` 底下的小工具、產生候選、定時 commit 產出的 Markdown 進 `content-hub/daily-tools/`；content-hub 只接收成品，不跑抓取邏輯

### daily-tools 呈現方式

- **GitHub Pages**（沿用既有作品集站 Albert-Pyahaw）：做一個輕量「工具圖鑑」頁，每個工具一張卡片（縮圖＋一句話痛點＋連結），比純 Markdown 列表更有瀏覽感
- **YouTube Shorts**：60 秒內展示工具操作畫面，日更量產成本最低，適合「每天一支」的節奏
- GitHub／`daily-tools/` 本身仍當 SEO 錨點與 NotebookLM 音訊來源，這層不變

### 吸引觀眾注意力的原則

- 標題走**痛點／成果導向**，不走功能描述（例如「這支工具幫我省了 2 小時排查」優於「XX 監控工具介紹」）
- **前 3 秒先給結果**（Before/After 畫面），原理放後面
- **固定時間發布**做成儀式感，05:40 本身就可以變成品牌記憶點（例如「晨間工具站」）
- **縮圖／封面統一視覺模板**（同色系、同字體），累積出一致的品牌辨識度
- 短影音為主、長影音為輔，短影音負責導流
- **結尾留互動鉤子**（提問、邀請留言分享你在用的工具），拉高互動率
