# C-State 待機功耗記錄工具組：一個測試需求、三代跨平台實作

一套針對「CPU 待機（Idle）／壓力（Stress）情境下的 C-State、頻率、溫度」量測需求，橫跨 Linux（Bash + Zenity GUI／TUI、Python 跨平台版）與 Windows（PowerShell）三條技術路線反覆迭代出來的工具組，附帶對應的 BIOS 設定對照表，用來驗證伺服器在「最大化功耗率」與「最低化功耗率」兩種極端情境下的行為是否符合預期。

## 這是什麼

這不是單一支腳本，而是同一個量測需求在不同時間點、不同平台上長出的三套獨立實作：

- **Linux Bash 版（`cstate_logger_gui.sh` v1.0.0，2025-10-02）**：鎖定 RHEL 9.x / SLE 15 SPx / openSUSE Leap 15.x，用 `turbostat` 定時採樣，首頁提供 `zenity` GUI（無 GUI 環境自動退回 TUI）四選一：1H Idle、12H Idle、1H Stress、Custom。
- **Linux Python 跨平台版（`cstate_logger_gui_v1.3.py`，2025-11-13）**：同時支援 Linux（優先用 `turbostat`，缺少則退回 `psutil` 或直接讀 `/proc`）與 Windows（`psutil`），單一 `.py` 檔案即可在兩個平台上跑，自動偵測 OS 並決定輸出路徑（root 執行時走 `/root/Documents`，一般使用者走 `~/Documents`）。
- **Windows PowerShell 版（`cpu_idle_state_win.ps1` v3.2.7 + `cpu_logger_gui_integrated.ps1`）**：獨立於 Python 版之外的另一條 Windows 專用路線，輸出結構更完整，包含 Chart.js 折線圖與可調式門檻設定。
- **BIOS 設定對照表（`BIOS set info.txt` + 兩張截圖）**：明確標註測「最低功耗率」要開 CPU C States、Package C State Limit 設 Auto 或 C6（甚至到 C10）；測「最大功耗率」則要讓 CPU 保持在 C0，兩種情境的 BIOS 選項完全相反，附截圖對照。

Linux 端資料夾裡還留著 SUSE／RHEL 各版本的舊腳本（`suse15sp6_cstate_v1/v2.sh`、`rhel9_later_v2/v3.sh`）與離線安裝套件包（`stress-ng` 與 `lksctp-tools` 的 rpm 檔），是為封閉機房、無外網環境準備的離線依賴安裝方案。

## 為什麼值得關注

伺服器功耗驗證有個常被忽略的前提：**BIOS 設定錯了，量測數字再精準也沒意義**。這套工具把「量測腳本」跟「對應該開哪個 BIOS 選項」綁在一起交付，而不是只丟一支腳本讓人自己猜該怎麼配置環境——`Input sop.txt` 裡甚至留了一段測試員實際操作紀錄（選 Non-idle 模式、間隔 60 秒、記錄 720 次）加上一句「請修 Log 的正排版」的手寫備註，看得出來這是從真實測試現場反覆修正出來的工具，不是憑空設計的範本。

更值得玩味的是它的**演化路徑**：同一個量測需求先在 Linux 上做出 Bash+Zenity 版，一個月後改寫成 Python 跨平台版試圖一次涵蓋兩個 OS，同時間 Windows 端又獨立發展出功能更完整（含 Chart.js 圖表、CSV/JSONL 輸出、可調門檻）的 PowerShell 專版並存至今。這種「沒有強行統一成一套架構，而是讓工具依平台分頭長出最適合的形態」的做法，反映了 SIT 現場工具開發的真實樣貌：先求可用、再求好用，跨平台一致性讓位給各平台上最快能用的方案。

## 技術重點（實際讀過原始碼）

- **單次採樣避免卡住**：Bash 版用 `turbostat --interval 1 --num_iterations 1` 每輪只取一筆，而不是讓 `turbostat` 常駐輪詢——這樣設計是為了讓每次採樣间隔可以自由控制（例如 12H Idle 情境是每 1800 秒採一次，而非讓 `turbostat` 自己維持長時間背景輪詢），降低長時間測試中腳本卡死或吃滿資源的風險。
- **三檔輸出設計（Albert Style）**：Bash 版每次測試會同時產出 Raw（完整 `turbostat` 原始輸出）、TXT（`Albert_Overview.txt` 關鍵資訊總表：OS/Mode/Interval/Repeats/Total/Start/End）、HTML（深色主題總覽頁，節錄第一筆採樣做預覽）三種格式，分別對應「原始證據」「快速總覽」「分享用報告」三種不同的閱讀情境。
- **Windows 版的門檻分級機制**：`thresholds.json` 讓 stress 模式與 idle 模式各自有獨立的 fail/error/critical/pass（或 info/warn）門檻值（例如 idle 模式 CPU 使用率超過 50% 就標記 critical），HTML 報告用 Chart.js 畫出 AvgCPU 隨時間變化的折線圖，並同時輸出 CSV／JSONL 供後續用 Excel 或程式二次分析，這是 Bash 版所沒有的資料再利用設計。
- **OS 自動偵測與路徑決策**：Python 版讀取 `/etc/os-release` 取得 `ID`／`VERSION_ID`／`PRETTY_NAME`，Windows 端則用 `os.sys.getwindowsversion()`；再依「是否為 root」決定輸出根目錄落在 `/root/Documents` 或使用者家目錄，避免不同權限下寫入失敗或誤寫到系統目錄。
- **離線環境的依賴補完**：README 明確寫出 SLE/Leap 無網路環境下要用 PackageHub POOL/UPDATES ISO 建立離線 repo，或直接用 rpm 離線安裝 `stress-ng`；並點名 `stress-ng` 若透過 rpmfind 離線安裝會缺 `libsctp.so.1`，需額外裝 `lksctp-tools`——這種踩過坑才知道的依賴細節，manifest 檔名完全看不出來，是實際下載讀取原始碼跟安裝套件包後才確認的。

## 可延伸應用角度

- 可以跟先前收錄的 NVQual、SUSE15SPX 工具箱放進同一條「硬體驗證工具」系列，這支剛好補上「功耗／C-State」這個切角，三支影片合起來能涵蓋一台伺服器上市前驗證的多個維度。
- BIOS 設定對照表跟腳本綁在一起交付這個做法本身值得單獨拆一支短片：示範「量測工具」與「量測前置條件」如何用一份文件同時交代清楚，避免只教工具操作、卻讓人測出不可信數據。
- 「同一個需求、三種平台各自演化出不同形態的實作」是一個很好的技術管理／工具設計案例，可以做成偏工程文化／方法論的影片，討論何時該投資做跨平台統一工具、何時該放手讓各平台各自最適化。
