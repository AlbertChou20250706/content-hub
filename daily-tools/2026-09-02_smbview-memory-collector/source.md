# SMBVIEW Memory Collector — 把 Fujitsu Primergy 伺服器讀 DIMM SPD 這件「會把自己鎖死」的事，做成兩段式自動化

> 來源路徑：`SMBVIEW_verify_Memory_for_SCD/`（Python/`albert_smbview_memory_v3.5.py` 為目前主線，`Preivous/` 保留 v1.1 → v3.2c → v3.3 → v3.4 完整版本歷史，`tools/` 內附 Fujitsu 原廠 `SMBVIEW64` v2.85 與 `Pc_Ident64` v4.07 二進位工具與官方說明文件）
> 設計者：Albert（SIT / MiTAC Computing）

---

## 這是什麼

一支包在 Fujitsu 原廠工具 `SMBVIEW64` 外面的 Python 封裝腳本，用來在 Linux 上自動收集 Primergy 伺服器（RX2530M8 / RX2540M8 / RX4770M8 等機型）每一根記憶體模組的 SPD（Serial Presence Detect）資料——廠牌、Part Number、序號、製造日期、頻率、溫度感測讀值、checksum 是否正常——並把原廠工具吐出的純文字 `.SCD` 檔轉成看得懂的 HTML／TXT／JSON 報表。核心價值不是「呼叫一個工具讀資料」，而是把「開製造模式 → 重開機 → 收資料 → 關製造模式」這一整套原廠沒有自動化、而且中間任何一步做錯就會讀到全部 `NotPresent` 的手動流程，收斂成一支腳本、兩個明確的執行階段。

## 為什麼值得關注

Fujitsu 原廠文件（`SMBVIEW_HELP.TXT`）寫得很清楚：RX2530/RX2540/RX4770 這幾個機型要讀 SPD，必須先用 `Pc_Ident64 flags.w=manuf` 把系統板切到「製造模式」，而且**一旦寫入 memory mark，SPD 就讀不到，必須重開機才能再讀**——這是一個「寫入動作會讓自己失效、且失效期橫跨一次 reboot」的原廠限制，沒有任何原廠腳本幫你處理這中間的狀態銜接。從版本歷史可以看到作者對這個限制的理解是逐步加深的：v3.1 的做法是偵測到 `0x3D Manufacturing mode is FALSE` 就切維護模式重試、寫 manuf、再讀一次、收尾自動 `flags.c=manuf` 關掉——想在同一次執行內做完整套流程；但這個「自動關回去」的收尾恰好撞上原廠那條「寫入後読不到、要重開機」的限制，容易造成關閉動作本身不完整生效。到了 v3.4／v3.5，設計整個改成**兩段式**：Phase 1（`enable`）只負責寫入製造模式並自動 `reboot`；Phase 2（`collect`）在重開機後執行，靠一個寫在 `~/.../PtuLog/SMBVIEW/.v3_5_state.json` 的狀態檔比對開機時間（`/proc/stat` 的 `btime`）來確認「真的已經重開過」，才會去收資料，不會在同一次開機內重複觸發製造模式切換。這種「先切一刀承認問題無法在單次執行內解決，用持久化狀態夾住一次 reboot」的設計轉折，是這份素材裡最值得拆解的部分。

## 技術重點（實際看過下載下來的內容）

- **用開機時間戳記而不是旗標檔判斷「有沒有真的重開機」**：`get_boot_btime()` 優先讀 `/proc/stat` 的 `btime`（開機時間的 Unix timestamp），讀不到才退回用 `/proc/uptime` 反推。`enable` 階段把當下的 `btime` 存進狀態檔；`collect`／`auto` 階段執行時重新讀一次目前的 `btime`，只有「目前值大於狀態檔存的值」才判定為「已完成重開機」，進而跳過重複寫入製造模式那一步。這比單純「檢查某個 flag 檔存不存在」更可靠，因為 flag 檔不會因為使用者手動重開機而自動更新語意，但 `btime` 一定跟著硬體重開機變動。
- **`Pc_Ident64` 的命令列語法有多種大小寫變體，腳本用「探測」而非「猜一種硬寫死」**：`pc_ident_args()` 針對 read/write/clear 三種操作各列出 5 種可能寫法（`flags.r=manuf` / `Flags.R=manuf` / `-flags.r=manuf` / `-Flags.R=manuf` / `-Flags R=manuf`），`probe_pc_ident_seed()` 逐一嘗試直到 `rc==0`，把成功的那個「種子」語法快取起來，之後 write／clear 操作用字串取代同一個種子裡的字母（`map_arg()`），確保三個操作用的是同一套機型剛好吃得下的語法格式，而不是各自硬編一種格式賭運氣。
- **`Pc_Ident64` / `pc_ident64` 大小寫檔名相容性是踩過的真實坑，回饋進了主程式**：README 明確寫「部分機型／套件在內部固定呼叫小寫 `pc_ident64`，但供應商常給大寫開頭的 `Pc_Ident64`；資料夾只有其中一個名稱可能導致 SPD 讀不到、整張表全是 `NotPresent`」。v3.5 主程式的 `ensure_pcident_alias()` 會在執行前自動檢查兩個檔名，缺哪個就幫哪個建 symlink 並補上執行權限——把一個原本要靠人工排查半天的環境問題，變成腳本自己修好再往下跑。
- **INI 讀取失敗會自動降級重試，而不是直接報錯中止**：`run_scd()` 若帶了 `--ini` 且原廠工具回報 `Read INI` / `ReadINIFile` 失敗，會自動改成不帶 `-INI` 的陽春模式重跑一次（除非顯式加 `--force-ini` 關掉這個 fallback）。這是從 v3.4 開始的行為，直接對應 README「疑難排解」章節記錄的真實錯誤訊息 `ERROR: Read INI file failed`。
- **GABI（廠牌自家的硬體存取驅動／服務）有 `auto/on/off` 三段式控制，而不是無條件啟動**：`--gabi auto` 只在維護模式（`flow_maint`）時才呼叫 `sobcontrol stop` 再 `start`；`off` 完全不碰，把決定權交給使用者，因為某些機型在非維護模式（`nonmaint`）下資料反而收得比較完整，不需要動用 GABI。舊參數 `--restart-gabi` 仍相容保留、等同 `--gabi on`，是漸進式重構、不砍舊介面的做法。
- **從 v1.1 到 v3.5 一路留著完整版本歷史**（`Preivous/For_Python/albert_smbview_memory_v3.2c` → `SMBVIEW_Memory_Collector v3.3_兩段式` → 上層 `Python/Provious/albert_smbview_memory_v3.4` → 現行 `Python/albert_smbview_memory_v3.5`），每個版本資料夾都各自帶一份對應的 quickstart README，等於是一份「原廠工具的坑，如何一版一版被摸清楚並寫進自動化邏輯」的活紀錄，跟同系列其他工具（`驗證WMI`、`SUSE15SPX` 的 SUT Toolkit）一樣是沒有用 Git、但保留了完整檔案級版本歷史的個人習慣。

## 可延伸應用角度

- **「單次執行做不完的操作，用持久化狀態卡住一次 reboot」這個設計模式**很適合獨立做一支影片，用 v3.1（同次執行內硬做完）到 v3.5（承認做不到、拆兩階段用 btime 比對）的演進當教材，這種模式不只適用於硬體維護模式，任何「改設定要重開機才生效」的自動化場景都能套用。
- **`Pc_Ident64`／`pc_ident64` 大小寫踩坑 + 探測式命令列語法適配**這兩點適合剪成「包裝原廠二進位工具時常見的環境相容性地雷」短片，跟已介紹過的 `SUSE15SPX` SUT Toolkit（同樣是包裝廠牌工具、處理環境差異）放在同一個系列裡對照。
- **INI 失敗自動降級重試、GABI 三段式開關**這種「操作失敗不中止、而是降級到更保守模式重試」的容錯設計，可以跟其他強調錯誤處理的既有素材（例如驗證類腳本）一起做成「維運腳本的容錯設計」主題合輯。
