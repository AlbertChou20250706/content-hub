PTU_CPU_Verify（Albert Style Python 版 v1.3.1）是一支包在 Intel 官方 PTU/PTAT 壓力測試工具外面的 Python 外殼：自動抓 PTAT 執行檔、開 turbostat 記頻率、算 RAPL 平均功耗，還內建「PTU 跑不動就退到 stress-ng、再不行就用 `yes` 硬灌」的三段式降級保底機制，跑完直接生出一份純前端 HTML 頻率趨勢報告，搭配一支零依賴的 Tkinter GUI 殼可以點按操作。

## 為什麼值得關注

在硬體驗證／SIT 實驗室裡，CPU 長跑壓力測試（1h/6h/12h/24h burn-in）是每天都要做的事：跑 serverlab 制式 100% 全核、或指定 AVX2／AVX-512 指令集施壓，看頻率會不會掉、功耗曲不曲線正常。問題是 Intel 官方的 PTU/PTAT 工具不是每台機器都能順利跑——可能缺驅動（`/dev/ptusys` 不存在）、可能這台系統就是裝不起來。如果驗證腳本一遇到 PTU 跑不起來就整個中止，值班工程師就要手動介入換工具重跑，長跑測試的自動化排程也會因此斷掉。這支工具把「保證能跑完」這件事寫死成程式邏輯，而不是靠人在旁邊盯著換指令。

## 技術重點

實際下載並讀過 `PTU_CPU_Verify.py`（294 行）與 `PTU_CPU_Verify_GUI.py`（214 行）之後，重點如下：

- **三段式降級**：主流程先試 PTU/PTAT（`build_ptu_cmd()` 依 profile 組出 `-ct 3/4/5`），跑完 `rc!=0` 就自動轉 `stress-ng --cpu-method matrixprod`，這步也失敗的話最後用 `yes > /dev/null` 開滿 `nproc()` 個進程硬灌 CPU，並直接把 `rc` 設為 0 收尾——確保不管環境缺什麼工具，這次驗證「一定會有一個結果」，不會卡在中途。
- **自動偵測與相容處理**：`autodetect_ptu()` 依序找 `PATH` 裡的 `ptat`/`ptu`，找不到再退到 `/opt/intel/ptu/bin/ptu` 等固定路徑；`id_flag_for()` 判斷檔名是 `ptat` 開頭且系統沒有 `/dev/ptusys` 時，自動幫命令加上 `-id` 旗標，這是作者從實際踩過的環境差異裡歸納出來的相容邏輯，不用使用者自己記。
- **量測與報表**：用 `turbostat --interval 1 --num_iterations {duration}` 取樣，`parse_trend()` 挑出 `Avg_MHz`/`Bzy_MHz` 欄位算逐秒平均；功耗則直接讀 `/sys/class/powercap` 底下所有 `energy_uj`，前後兩次讀值相減除以秒數換算平均瓦數（RAPL），沒有安裝任何量測 SDK。`make_html()` 甚至沒有引用任何圖表函式庫，是手刻一段 vanilla JS 直接用 SVG `<path>`/`<line>`/`<text>` 畫頻率趨勢折線圖，整份 `Albert_Overview.html` 就是一個自包含檔案。
- **Governor 切換有還原**：跑之前若選 `performance` 且有 `cpupower`，會先切 `performance` governor 並記下 `restore_gov=True`，測試結束後主動切回 `ondemand`——避免長跑測試「順手」把系統效能模式永久改掉。
- **GUI 只是一層殼**：`PTU_CPU_Verify_GUI.py` 純用內建 Tkinter（不需額外套件），本身不執行測試邏輯，只是把欄位值（Duration/Load/Governor/Cores/Profile 等）組成環境變數，`subprocess.Popen` 呼叫同資料夾的 `PTU_CPU_Verify.py`；使用者填的參數會存到 `~/.ptu_cpu_verify_gui.json`，下次開啟自動帶回，並有「View processes」按鈕跑 `pgrep -af 'ptat|turbostat|stress-ng|yes'` 讓人確認測試真的在跑。
- **README 裡藏了真實踩坑紀錄**：文件特別寫了一段「無網路環境用 RHEL ISO 安裝 tkinter」的手法（`mount -o loop` 掛 ISO、直接用 `dnf install /mnt/AppStream/Packages/python3-tkinter-*.rpm`），這是內網無法連外的驗證機房才會遇到的真實限制，也是這份素材裡最有「實戰感」的細節。

## 可延伸應用角度

- 適合做成「怎麼幫官方 CLI 工具包一層容錯殼」的設計案例：三段式降級（PTU → stress-ng → yes soaker）是個很好教材，示範如何讓自動化測試「保證有結果」而不是遇到環境差異就整個失敗。
- 可以搭配既有的 SIT／硬體驗證系列（例如先前介紹過的 RAID CC、SUT Toolkit、SMBVIEW 系列），組成一集「機房裡的容錯設計模式」，比較不同工具處理環境不確定性的手法。
- 手刻 SVG 折線圖（不依賴 Chart.js 等函式庫）這段本身可以獨立拉出來做一支「零依賴前端視覺化」的短影片，示範怎麼用不到 50 行 JS 畫出一張可用的趨勢圖。
