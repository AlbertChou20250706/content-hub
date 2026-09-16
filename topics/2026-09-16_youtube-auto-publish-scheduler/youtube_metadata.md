# YouTube Metadata｜YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知規劃

> 影片尚未錄製／上架，以下為草稿內容，供之後正式錄製時參考與調整。目前仍在技術規劃階段，自動化 repo 尚未建立、尚未完成實作。

## 標題（實際發布用，≤ 100 字元）

YouTube 影片排程自動發布系統：GitHub Actions 批次清空佇列 + LINE 通知

## 說明欄（Description）

```
📖 本集重點
1. 溫馨提醒請記得開啟字幕
2. 本機電腦常態關機，改用 GitHub Actions 雲端排程做「YouTube 影片延遲公開」
3. 完整流程：讀取待發布播放清單 → 批次逐支轉為 public → 更新已發布清單 → LINE Messaging API 彙總通知 → commit 回 repo
4. 為什麼從「每次發 1 支」改成「每次觸發批次清空佇列」
5. YouTube OAuth 沒切到 Production 的話，Refresh Token 7 天就會失效的坑
6. 自動化程式碼與內容紀錄分兩個 repo 的原因與分工方式

📚 延伸閱讀
完整技術規格與程式碼：<待補，自動化 repo 建立後回填>
完整文字紀錄：<content-hub 該篇 topics 資料夾連結，待補>

🔔 追蹤 Albert
🎬 YouTube：https://www.youtube.com/@12100105aym
💻 GitHub：https://github.com/albertchou20250706

🎬 本集製作
監製・企畫・錄音｜Albert.Chou
轉錄・校對・字幕・影片｜AI 自動化工作流
（Whisper 語音轉錄 × Claude 校對整理 × NotebookLM 影片生成）

完整工作流與腳本已開源，歡迎交流。
#YouTube自動化 #GitHubActions #ClaudeCode #LINEBot #排程發布
```

## 時間戳章節（Chapters）

00:00 開場：為什麼要做 YouTube 排程自動發布
00:00 為什麼放棄本機排程器，改用 GitHub Actions
00:00 架構總覽：批次清空佇列的四段流程
00:00 YouTube OAuth Production 設定的關鍵坑
00:00 LINE Messaging API 通知設計
00:00 結語與後續規劃

> 待正式錄製、剪輯完成後依實際時間點填入。

## 標籤（Tags，逗號分隔，≤ 500 字元）

YouTube自動化, GitHub Actions, YouTube Data API, LINE Bot, LINE Messaging API, 排程發布, cron, workflow_dispatch, ChouAP.Cloud, OAuth

## 分類（Category）

Science & Technology

## 縮圖文字（Thumbnail Text，≤ 6 字為佳）

YT 自動排程發布

## 播放清單

<待正式錄製後確認是否歸屬既有播放清單，或另開新播放清單>

## 卡片／結束畫面（Cards / End Screen）

- 卡片：（未確認）
- 結束畫面推薦影片：（未確認）

## SEO 關鍵字檢查

- [ ] 標題含核心關鍵字
- [ ] 說明欄前兩行含核心關鍵字
- [ ] 標籤涵蓋同義詞／相關搜尋詞
