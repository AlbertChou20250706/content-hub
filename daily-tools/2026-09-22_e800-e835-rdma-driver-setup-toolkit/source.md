E800_setup 是一套針對 Intel Ethernet 800 系列網卡（E810 / E830 / E835）的驅動部署與 RDMA 效能驗證工具組，核心是一支自動化安裝腳本 `Intel_E835_RDMA_Setup.sh`（README 裡還規劃了效能驗證與即時監看兩支腳本，但目前資料夾中實際落地、可下載到的只有這支安裝腳本本身）。

## 為什麼值得關注

Intel 800 系列網卡的驅動安裝流程本身就有不少地雷：ice base driver 跟 irdma RDMA driver 必須成對、依固定順序（先 ice 後 irdma）安裝，版本要對齊，路徑不能有空白或冒號（Intel 官方 Makefile 會直接拒絕編譯）。這些細節單靠人工照著官方 README 做，很容易在某個環節卡住又抓不到原因。這支腳本把 SIT 測試站台實際踩過的坑，逐一寫成程式碼裡的防呆與自動修正邏輯，而且是在同一台 MITAC B8056 平台上實測過 200GbE 環境（E835-CCQDA1M 單埠、E835-CQDA2M 雙埠皆已驗證安裝流程）才定案的版本。

## 技術重點

實際下載並讀過 README 與腳本原始碼後，幾個值得注意的設計：

- **版本履歷本身就是一份真實 bug 修復案例集**：腳本檔頭用 `REM ==========` 分段記錄了 v1.0.0 到 v1.2.2 每一版的根因與修法，不是空泛的「修了一些 bug」。
- **`verify_rdma()` 的誤判修正（v1.2.2）**：舊邏輯用 `grep -q 'irdma'` 比對 `ibv_devices` 輸出來判斷裝置是否存在，但 RoCEv2 模式下裝置名稱是 `rocepXXXsY`（例如 `rocep225s0f0`），完全不含「irdma」字串，導致機台明明兩張卡都正常運作、node GUID 也正常，腳本卻永遠回報 FAIL。修正後改成直接數 `ibv_devices` 表頭下方的裝置列數（`tail -n +3 | grep -c '[^[:space:]]'`），不管是 iWARP 的 `irdmaN` 還是 RoCEv2 的 `rocepXsY` 命名都能正確判斷。
- **`disable_cdrom_source()` 解決的是實體媒體提示、不是 y/n 問題（v1.2.1）**：部分機台 `/etc/apt/sources.list` 殘留 Ubuntu 安裝媒體留下的 `deb cdrom:...` 來源，apt 一旦認為套件可能來自光碟就會卡在「Media change: please insert the disc」提示——這是實體媒體提示而非 y/n 問句，`apt install -y` 的 `-y` 完全無法跳過，且腳本輸出經 `tee` 導管後 stdin 有時收不到即時輸入，畫面會無限重複同一句提示、CPU 卻是 0%，外觀上跟腳本卡死一模一樣。修法是在 `apt update`/`apt install` 前呼叫 `disable_cdrom_source()`，找到 `deb cdrom:` 開頭的行就先備份成 `.bak.<timestamp>` 再用 `sed` 註解掉。
- **dmesg Unknown symbol 偵測用時間切片而非全量比對（v1.2.0）**：早期版本抓 `dmesg` 最後 50 行找「Unknown symbol」，結果連 ice 驅動自己 depmod 過程留下的舊警告都被算進去，把正確安裝的 irdma 誤判成 CRIT/FAIL；修正後改成在 `modprobe irdma` 前先記錄 dmesg 行數快照，只掃描快照之後新增的行。
- **路徑鐵律**：README 明確要求 `~/E800_setup` 到 `Source/ice-2.6.7/` 這條路徑鏈路上不能有任何空白字元或冒號，因為 Intel 驅動的 Makefile 會主動檢查並拒絕編譯——這也是為什麼 v1.1.0 額外處理了「已解壓縮 vs 壓縮檔」兩種來源型態，避免使用者手動解壓縮到含空白的資料夾。

## 可延伸應用角度

- 適合做成「除錯思路 / Root Cause 分析」系列影片：三個版本修正（cdrom 媒體提示、irdma 誤判、dmesg 誤判）都是「表面現象跟真正原因完全對不上」的經典案例，很適合拿來講「畫面卡住不代表卡死」「FAIL 訊息不代表真的失敗」這類排查心法。
- 也適合搭配硬體驗測 / SIT 測試流程類內容，講解 RDMA over Converged Ethernet（RoCEv2）跟 iWARP 在驗證邏輯上的差異，用這支腳本的裝置命名判斷邏輯當範例。
- 可以延伸討論「基礎設施腳本要不要寫版本履歷」這個主題——這支腳本把每次修 bug 的根因和邏輯直接寫進檔頭注釋，本身就是一份可追溯的維運文件，值得拿來對比一般專案常見的「commit message 寫不清楚」問題。
