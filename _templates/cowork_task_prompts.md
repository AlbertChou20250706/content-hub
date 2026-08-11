# Cowork 交接 Prompt 範本｜content-hub 新主題

> 用途：每次要為 content-hub 新增一個主題（新的 SIT 驗證／自動化紀錄）時，
> 複製這份、填入實際值，貼給 Cowork／Claude Code 開新任務。
> 目的是每次交接都有完整背景，不用重新口述一次架構決策。

---

## 背景與目標

<這次要記錄的技術工作是什麼？跟 content-hub 現有主題的關聯？>

content-hub 定位：SIT 驗證與自動化實戰的公開紀錄庫，串接 GitHub → YouTube → NotebookLM 內容飛輪。
Repo：<content-hub GitHub 網址>

## 本次資料

- 主題 slug：`YYYY-MM-DD_<主題slug>`
- 分類：<SIT 驗證 / 自動化工具 / 踩坑紀錄>
- YouTube 影片狀態：<已發布連結 / 錄製中 / 尚未開始>
- 素材位置：<逐字稿／腳本／截圖等現有素材的路徑>

## 待執行任務（請逐步執行，每步完成後回報結果，等我確認再進下一步）

### Step 1｜建立主題資料夾

`topics/YYYY-MM-DD_<主題slug>/`，套用 `_templates/README_template.md` 填寫。

### Step 2｜YouTube 腳本／metadata（若影片尚未發布）

套用 `_templates/youtube_script_template.md`、`_templates/youtube_metadata_template.md`。

### Step 3｜社群貼文（影片發布後）

套用 `_templates/social_posts_template.md`。

### Step 4｜回填首頁索引

更新 content-hub 根目錄 `README.md` 的主題索引表格，補上這一筆（主題｜日期｜分類｜YouTube連結｜狀態）。

### Step 5｜更新發布紀錄

在 `_log/publish_log.md` 追加一筆本次發布紀錄。

### Step 6｜commit & push

## 完成後請回報

1. 主題資料夾實際路徑
2. 首頁索引表格新增的那一行內容
3. 是否有任何步驟因缺資料而跳過，跳過的原因
