# kdump-helper.sh：一支腳本搞定 RHEL/SUSE 系統的 kdump 健檢、crashkernel 設定與當機取證

`kdump-helper.sh` 是一支跨 RHEL-like（RHEL／Rocky／Alma／CentOS Stream）與 SUSE-like（SLES／openSUSE／SUSE 16）發行版的 kdump 健檢與設定輔助腳本，把「確認 kdump 服務有沒有啟用」「crashkernel 開機參數有沒有正確保留」「要不要手動觸發一次當機測試來驗證 vmcore 真的會產生」這三件 SIT 工程師在驗機、交機前必做但步驟瑣碎又容易漏掉的檢查，收斂成幾個明確的 CLI flag。

## 為什麼值得關注

kdump（核心崩潰傾印）是伺服器 RAS（可靠性）驗證的基本功，但實務上最痛的不是「知道要開 kdump」，而是「RHEL 系用 `grubby`、SUSE 系要手動改 `/etc/default/grub` 再跑 `grub2-mkconfig`，兩條路完全不同」，以及「改完 crashkernel 沒重開機，驗證時才發現沒生效」這類跨平台差異與遺漏。這支腳本把兩個發行版族系的處理邏輯都寫進同一支腳本裡，靠 `/etc/os-release` 自動判斷分支，工程師不用每次都去查兩套指令怎麼下，也不用在 SOP 文件裡分別列 RHEL 版和 SUSE 版步驟。

## 技術重點（實際讀過原始碼）

- **五種模式、單一入口**：`--status`／`--enable`／`--ensure-crashkernel`／`--restart`／`--test-crash`，靠 `parse_args()` 解析成單一 `MODE` 變數分派到對應函式，`do_enable` 甚至會依序呼叫「盡力安裝套件 → 確保 crashkernel → enable+start service → 跑一次完整 status」四個動作，把「從零到可用」壓成一條命令。
- **RHEL／SUSE 分岔處理是核心賣點**：`is_suse()` / `is_rhel_like()` 用 `/etc/os-release` 的 `ID`／`ID_LIKE` 搭配 regex 判斷家族；RHEL 路徑呼叫 `grubby --update-kernel=ALL --args=...`，SUSE 路徑則是直接用 `sed` 對 `/etc/default/grub` 的 `GRUB_CMDLINE_LINUX_DEFAULT` 做插入（動手前會先 `cp -a` 備份成 `.bak.<timestamp>`），再跑 `grub2-mkconfig` 重建設定檔，兩條路徑互不干擾。
- **冪等設計**：`do_ensure_crashkernel` 一開始就檢查 `/proc/cmdline` 現有的 `crashkernel=` 是否已存在，存在就直接跳過、明確標註「不會覆蓋既有設定」，避免重複執行腳本把設定越改越亂。
- **CLI + HTML 雙格式留痕**：`say()` / `run()` 這兩個工具函式把每一段輸出同時寫進純文字 log 與一份帶內嵌 CSS 的 HTML log（`html_section_open/close`），對需要附驗證紀錄給客戶或存檔的 SIT 場景很實用，不用另外截圖或整理報告。
- **`--test-crash` 走的是真崩潰**：直接 `sysctl -w kernel.sysrq=1` 後 `echo c > /proc/sysrq-trigger` 觸發真正的 kernel panic 來驗證 vmcore 是否落地，腳本刻意要求互動輸入 `YES`（或明確帶 `--yes`）才會執行，並在說明文件與程式碼裡都標註「僅限測試機、會立即重開機」。
- **雙語巧思**：對外訊息（CLI 輸出）全部英文，但程式碳裡的註解刻意寫成「日文漢字＋假名讀音」（例如 `メタ情報（めたじょうほう）`），是作者 Albert.Chou 這批工具裡少見、帶點個人風格的雙語筆記法，推測是同時練日文與寫技術文件的習慣。

## 可延伸應用角度

- 適合做成「RAS 驗證系列」的其中一集：對比 RHEL 與 SUSE 兩條 crashkernel 設定路徑的差異，用這支腳本的 `--status` 輸出當示範素材，直接展示同一支指令在兩種發行版上跑出不同分支邏輯。
- `--test-crash` 觸發到 vmcore 落地的完整流程，可以錄成一支「當機取證從觸發到分析」的實機示範影片，搭配 `show_vmcore_latest()` 找到最新 vmcore 的邏輯講解。
- CLI + HTML 雙格式留痕的設計，可以跟先前收錄的其他「零依賴單檔工具」（如 OT Calculator、RAID CC 驗證工具）放在同一個「工程師自製留痕/報告工具」系列裡做橫向比較，凸顯這批工具一致的「跑完就要有證據」設計哲學。
