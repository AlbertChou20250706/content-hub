Win2025 AutoLogin Toolkit 是一組針對 Windows Server 2025／Windows 11 的自動登入設定批次工具，把「改 Registry 讓伺服器開機直接進系統、不用手動打密碼」這件事，從一段容易打錯的 `reg add` 指令，包裝成四支各司其職、帶色彩提示與 Log 紀錄的 `.bat` 腳本。

## 為什麼值得關注

SIT／測試機房裡有大量伺服器需要無人值守重開機（跑穩定性測試、無預警斷電復電、遠端 KVM 不方便輸入密碼），手動登入是最常被忽略但又最麻煩的一環：Registry 路徑長、四個值要一次設對（`AutoAdminLogon`／`DefaultUserName`／`DefaultPassword`／`DefaultDomainName`），設定錯一個就會卡在登入畫面。這個工具包把「設定、批次部署、驗證、還原」四個場景都各自寫成一支腳本，而不是只給一段指令要人自己套用，是很典型的「把重複性 IT 維運工作腳本化」案例。

## 技術重點

實際下載並讀過六個檔案後，重點如下：

- **四支腳本各司其職**：`win2025_autologin_setup.bat` 是互動精靈（`set /p` 問使用者名稱／密碼／網域，預設本機網域是 `.`），`win2025_autologin_silent.bat` 是無互動版本（直接吃命令列參數，回傳 0/1 當作 exit code，方便被別的自動化腳本呼叫），`win2025_autologin_disable.bat` 專門移除 `DefaultPassword` 但保留 `DefaultUserName`（設計上刻意讓「關掉自動登入」跟「完全清除痕跡」分開），`win2025_autologin_verify.bat` 純讀取不寫入，用來做 CI 或部署後的健康檢查。
- **每支 setup/verify 腳本都有管理員權限檢查**（`net session >nul 2>&1` 判斷 errorlevel，不是靠 `whoami /groups` 或第三方模組）以及執行完自動寫 timestamp 命名的 `.log` 檔（`autologin_setup_YYYYMMDD_HHMMSS.log`），這代表這是真的被拿去多台伺服器上跑過、需要留稽核紀錄的工具，不是隨手寫寫的一次性腳本。
- **色彩化 CLI 回饋**：用 `color` 指令搭配自訂色碼（紅 0C=錯誤、綠 0A=成功、黃 0E=警告）讓 batch 腳本在終端機上有近似「測試框架」的視覺回饋，而不是純文字捲動，對於要盯著遠端 KVM 螢幕確認每一步是否成功的場景很實用。
- **README 本身就是一份風險揭露文件**：直接寫明密碼以明文存在 `HKLM\...\Winlogon\DefaultPassword`，任何本機管理員或備份檔都能讀到，並列出三種替代方案（Credential Manager／Group Policy 免 Ctrl+Alt+Del／TPM+BitLocker），附上使用前的安全檢查清單。這種「工具能用，但誠實告知風險與替代解法」的寫法，比很多網路上流傳的自動登入教學都更負責任。
- 隨附的 `AutoAdminLogon.ps1` 是最小可行版本（PowerShell 用 `Set-ItemProperty` 直接寫四個值），可以拿來跟四支 batch 腳本對照，示範同一件事用 PowerShell vs Batch 兩種寫法的差異。

## 可延伸應用角度

- 適合做成「IT 維運腳本化」系列：示範怎麼把一個大家都知道但嫌麻煩去做的 Registry 設定，變成有輸入驗證、有 Log、有還原路徑的正式工具，而不是丟一段指令了事。
- 可以搭配資安／機房管理角度的內容，講解明文密碼存在 Registry 的實際風險與三種替代方案的取捨（哪種適合開發機、哪種適合正式機、哪種需要 AD 網域）。
- 也適合拿 `AutoAdminLogon.ps1`（PowerShell 極簡版）跟四支 `.bat`（互動、無人值守、驗證、還原）做並排比較，講「同一個 Registry 操作，寫成臨時腳本 vs 寫成給別人用的工具，中間差了哪些東西」（權限檢查、輸入驗證、Log、色彩回饋）。
