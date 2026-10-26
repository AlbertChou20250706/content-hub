AVT_Install_Master_ubuntu.sh 是一支把「在 Ubuntu 上安裝 AMD Validation Toolkit（AVT）」整套流程自動化的 Bash 腳本：偵測網路、把官方只提供的 RPM 套件轉成 DEB、修復相依關係、驗證安裝結果，最後同時產出文字與 HTML 雙格式的安裝報告。

## 為什麼值得關注

AVT 官方只發行 RPM 與原生 tarball，沒有提供 DEB 套件，但驗證團隊的機器經常跑在 Ubuntu 上，於是每次要裝 AVT 時都得手動用 `alien` 轉檔、處理 `libOpenCL.so.1` 缺相依、再自己慢慢排查 `dpkg` 錨誤——是一個「廠商沒照顧到你的平台」的典型痛點。這支腳本把這整套手動排雷步驟寫成一個互動式安裝器，而且附上 `--dry-run` 先跑一遍不動手，再搭配完整分類的故障排除手冊，等於把一次性的「怎麼裝」知識固化成可以重複使用、可以交給別人執行的工具。

## 技術重點

實際下載並讀過資料夾內容（README、MD 子資料夾說明文件、以及 835 行的腳本本體）後，值得注意的設計：

- **RPM → DEB 自動轉換**：`install_alien()` 先確認系統有沒有 `alien`，沒有就先 `apt install -y alien`，再用 `convert_rpm_to_deb()` 跑 `alien -d` 把官方 RPM（含 `ocl-icd` OpenCL ICD loader）轉成可以用 `dpkg` 直接裝的 DEB，解決「官方只給 RHEL 系套件」的平台落差問題。
- **五個可組合的命令列旗標**：`--skip-network`、`--skip-backup`、`--dry-run`、`--force`、`--verbose`，對應腳本裡的 `SKIP_NETWORK_CHECK`／`SKIP_BACKUP`／`DRY_RUN`／`FORCE_INSTALL` 旗標變數，讓同一支腳本可以適應「已知網路檢查會誤判」「要先模擬跑一次」等不同情境，而不用改腳本本體。
- **裝前備份、裝後驗證形成一個完整迴圈**：`backup_system()` 在安裝前用 `dpkg -l` 存一份套件清單快照，`verify_installation()` 在裝完後用 `dpkg -l | grep` 反向確認目標套件真的進去了；README 裡還教怎麼拿安裝前後的備份檔案做 `diff` 比對，形成「裝之前有底、裝之後有驗」的操作習慣。
- **雙格式日誌輸出**：`init_logging()` 同時寫文字 log 跟呼叫 `generate_html_log()` 產生帶 CSS 樣式的 HTML 報告，方便不同情境下查閱（文字 log 適合 `grep` 搜錯誤，HTML 報告適合直接丟給非技術人員看結果）。
- **README 誠實標註了作者沒驗證過的部分**：文件裡提到 `AVT_Help_Creator.bat` 是「用途依檔名判斷為說明文件產生器，本說明未確認其內容」，這種「不確定就寫不確定」的文件風格，跟同資料夾裡用日文註解（附讀音）搭配英文輸出訊息的雙語設計一樣，都是這份文件可信度高、不是隨便拼湊的佐證。
- **NDA 與保密標示**：`AVT_Linux_2.8.57/` 底下的安裝套件與說明文件（PDF、ReadMe_NDA.txt）都帶有 NDA 字樣，README 也明確提醒「請勿對外散布」——這點在之後做成內容時需要避開展示受保密約束的實際套件內容，只能談安裝腳本本身的設計。

## 可延伸應用角度

- 適合做成「幫廠商工具補一塊缺口」系列：示範當官方套件格式跟你的作業系統不匹配時，怎麼用 `alien` 這類轉換工具搭建橋樑，而不是每次都手動踩雷。
- 可以搭配「安裝腳本的防呆設計」主題，拿 `--dry-run`、裝前備份、裝後驗證這三個環節當範例，講解一支好的自動化安裝腳本該具備哪些安全網。
- 也適合跟之前介紹過的《SIT Master Toolkit》串成系列——那支工具的「AVT Workflow Manager」管理的是驗證流程本身，而這支腳本解決的是「怎麼把 AVT 裝上去」這個更前置的步驟，兩者可以組成「從安裝到驗證」的完整故事線。
