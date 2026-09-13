# 免打包 Electron 也有桌面體驗：用純 HTML + Edge 啟動器打造 OT Calculator

> 工具紀錄：單檔離線加班時數試算工具，從純網頁工具到正式上架 Microsoft Store 的完整過程

| 項目 | 內容 |
|---|---|
| 日期 | 2026-09-05 |
| 分類 | 自動化工具 |
| 狀態 | 已發布 |
| YouTube | https://www.youtube.com/watch?v=JzPmid-098M |
| NotebookLM | |

## 背景

製造業與 SIT 團隊常見的加班時數計算，公司之間的規則差異很大（午休是否扣除、晚餐後幾點起算、進位規則捨去或四捨五入到 0.5h／1h），手算或用 Excel 每次都要重新對規則，容易算錯也很花時間。想要一個能把規則參數化、輸入打卡時間就自動換算的工具，而且不想為了一個小工具動用 Electron 這種重量級打包方案。

## 做了什麼

1. 用純 HTML + vanilla JS 打造單檔、離線可用的加班計算工具，把公司規則（標準工時、午休扣除、晚餐門檻、進位方式）全部參數化
2. 設計 PowerShell 啟動器，偵測 Edge 瀏覽器並用 `--app=` 參數無框啟動，做出接近原生桌面 App 的視覺體驗，完全不用打包 Electron
3. 加上防抖動保護機制（`GUARD` 除錯開關，搭配 smallDrift／hugeJump 門檻），避免使用者連點按鈕造成畫面閃爍或資料錯亂
4. 版本從 v1.0 一路演進到 v1.3.0，陸續補上三語系切換、週彙總看板、CSV 匯出、六種主題
5. 後續補上 PWA `manifest.json` ／ service worker，透過 PWA Builder 打包成 MSIX，正式以 **AlbertBiahal** 這個 Publisher 上架 Microsoft Store

## 關鍵發現／重點

- 純 HTML + Edge `--app=` 啟動器，就能做出不輸 Electron 的桌面體驗，不需要額外的打包引擎、也沒有安裝檔膨脹的問題
- 個人工具很容易出現「功能領先、部署落後」——邏輯先做對，架設／上架流程往往是後補的，這支影片本身就記錄了這個真實的工程演進過程
- 要把純網頁工具送上 Microsoft Store，不一定要走 Nuitka／原生編譯這條路：**PWA Builder** 對這類單檔 HTML 工具是更輕量、免費的打包路徑
- 上架過程有個容易忽略的坑：即使 IARC／隱私問卷誠實選「否」（不存取個資），只要套件宣告了 `runFullTrust` 這類受限功能，Partner Center 仍會強制要求提供隱私政策 URL 才能通過

## 行動項（若有後續待辦）

- [ ] 影片說明欄補上 Microsoft Store 下載連結（https://apps.microsoft.com/detail/9NKGTSLTN8RN），目前只有 YouTube／GitHub 個人首頁連結
- [ ] GitHub 連結改指向 `OT_Calc_Launcher` repo 本身，而不是帳號首頁

## 延伸資源

- 相關 repo：https://github.com/AlbertChou20250706/OT_Calc_Launcher
- Microsoft Store：https://apps.microsoft.com/detail/9NKGTSLTN8RN
- 線上體驗（GitHub Pages）：https://albertchou20250706.github.io/OT_Calc_Launcher/OT_Calculator.html

---
*此篇為 [content-hub](../../README.md) 系列紀錄之一。*
