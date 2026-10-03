# setup_console_autologin.sh — Ubuntu Server 純文字 console 自動登入設定工具

一支互動式 bash 腳本，讓沒有圖形介面的 Ubuntu Server 開機後直接自動登入指定使用者的 console（tty），不需要手動改設定檔、不需要記任何命令列參數。

## 這是什麼

腳本執行後會問三個問題：要自動登入哪個使用者（預設 `root`）、套用在哪個 tty（預設 `tty1`）、最後確認是否套用。回答完就自動寫好設定、重載 systemd、重啟對應的 getty 服務。整支腳本大約 300 多行，日文夾雜英文的版本記錄與註解風格明顯是作者一貫的個人習慣，內建完整的 log 分級（INFO/PASS/WARN/FAIL/CRIT/ERROR）與彩色輸出。

## 為什麼值得關注

實驗室或測試機房裡，機器常常要重開機、換版本、跑壓力測試，每次開機後都要接 iKVM 或 BMC console 手動輸入帳密才能看到系統狀態，是很煩的重複勞動。這支工具解決的正是這個「機房內受控環境」的場景——README 也明確把「對外/正式環境」列為不建議使用，安全邊界劃得很清楚：只影響 console/getty 登入，不影響 SSH。

## 技術重點

- 沒有走「直接改 `/lib/systemd/system/getty@.service`」這種粗暴做法，而是在 `/etc/systemd/system/getty@<tty>.service.d/` 建立 systemd drop-in override（`autologin.conf`），好處是套件升版不會覆蓋設定，移除也只要刪掉這個 override 檔案。
- drop-in 內容分兩行：先用空的 `ExecStart=` 清掉原本的啟動指令，再用 `ExecStart=-/sbin/agetty --autologin <user> --noclear %I $TERM` 重新定義——這個「先清空再覆寫」的寫法是 systemd override 常見但容易忽略的細節。
- 使用者輸入驗證會動態列出 `/etc/passwd` 裡 UID ≥ 1000 且 shell 不是 `nologin`/`false` 的帳號當提示，並檢查 tty 格式必須是 `tty` + 數字（明確排除 `ttyS0` 序列埠、`tty0`）。
- 若偵測到 `gdm3`/`gdm` 正在跑（代表其實是桌面版），會印警告提示改用 GNOME 的圖形化自動登入設定，但不會擋執行。
- 寫入設定前若偵測到舊的 override 檔案，會先備份成帶時間戳的 `.bak` 檔，撤銷或還原都有對應的還原指令，README 裡的「還原」章節寫得很完整。
- 全程 log 到 `/root/Documents/setup_console_autologin_<時間戳>.log`，方便事後稽核誰在什麼時候改過哪台機器的 console 登入設定。

## 可延伸應用角度

適合搭配既有的機房自動化／SOP 系列做成一支短影片：用互動式問答降低操作門檻 vs. 直接改設定檔的風險，是很好的「工具設計取捨」案例。也可以跟同系列的 Windows 版本（Win2025 AutoLogin Toolkit,2026-09-27 已收錄）並列比較，一支講 Linux 的 systemd drop-in 機制、一支講 Windows 登錄檔機制，剛好構成跨平台自動登入設定的對照組。
