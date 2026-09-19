`Installing windows Hyper-v`（核心腳本 `install-hyperv.ps1`、`create-hyperv-resources.ps1`、`create-hyperv-paths-and-vm.ps1`）是一組把 Windows Server 從「乾淨系統」推進到「能跑 VM 的 Hyper-V Host」全程腳本化的三支 PowerShell 工具，分別對應三個階段：啟用 Hyper-V 角色並重開機、建立虛擬交換器與示範 VM、設定 Host 層級的 VM／VHD 預設路徑與 NTFS 權限再建立示範 VM。

## 為什麼值得關注

手動建置 Hyper-V Host 有幾個容易踩雷、但每次都要重做一次的環節：安裝完 Hyper-V 角色後一定要重開機，`New-VM` 之類的指令在重開前根本不存在，很多人第一次裝就卡在這步；建立虛擬交換器要綁對實體 NIC 名稱，環境一多容易選錯；如果把 VM 設定檔和 VHDX 硬碟檔分別放到不同磁碟（常見的效能隔離作法），Hyper-V 服務（以 SYSTEM 身分執行）對非預設路徑不見得有寫入權限，「存取被拒」的錯誤訊息往往讓人抓不到頭緒是權限問題。這套腳本把這些容易漏掉的細節（重開機時機、NIC 綁定、路徑權限）直接寫進自動化流程裡，不用每次建置測試機都重新想一遍。

## 技術重點（實際讀過原始碼）

- **三支腳本開頭都用同一段管理員權限檢查**：`[Security.Principal.WindowsPrincipal]...IsInRole("Administrator")`，不是管理員就直接 `Write-Error` 並 `exit 1`，整組工具在「防呆」這件事上風格一致。
- **`install-hyperv.ps1` 只有 14 行，但把最容易忘記的一步寫死進去**：`Install-WindowsFeature -Name Hyper-V -IncludeManagementTools` 之後緊接著 `Restart-Computer -Force`，直接把「裝完角色一定要重開機，不然後面的 VM 指令會失敗」這個常見踩雷點變成腳本行為，而不是靠使用者自己記得。
- **`create-hyperv-resources.ps1` 走冪等設計**：用 `Get-VMSwitch -Name $SwitchName -ErrorAction SilentlyContinue` 和 `Get-VM -Name $VMName -ErrorAction SilentlyContinue` 先檢查是否已存在，存在就跳過、不存在才建立，代表這支腳本可以重複執行而不會炸出「已存在」的錯誤；VM 預設建成 Generation 2（支援 UEFI／Secure Boot），ISO 路徑抓不到時只印警告訊息、不會讓整支腳本失敗。
- **`create-hyperv-paths-and-vm.ps1` 是三支裡功能最完整的一支，處理了前一支沒管的權限問題**：把 VM 設定檔路徑（範例 `D:\Hyper-V\Virtual Machines`）和 VHDX 硬碟路徑（範例 `E:\Hyper-V\Virtual Hard Disks`）分開設定，對兩個資料夾各自跑 `icacls ... /grant "NT AUTHORITY\SYSTEM:(OI)(CI)F" /T` 和對 `BUILTIN\Administrators` 做同樣的遞迴授權——這正是解決上面提到「换路径后存取被拒」問題的關鍵一步；接著呼叫 `Set-VMHost -VirtualMachinePath ... -VirtualHardDiskPath ...` 讓路徑設定立即生效、不需要重開機；建立示範 VM 時改用 `(Get-VMSwitch | Select-Object -First 1).Name` 自動抓現有的第一個虛擬交換器，不像前一支腳本需要手動指定 NIC 名稱。
- **從下載下來的 `files.zip` 可以看出版本演進**：這個壓縮包（2025-12-03 的快照）裡只有 `install-hyperv.ps1` 和 `create-hyperv-resources.ps1` 兩支，代表工具原本只處理「裝角色」和「建交換器＋VM」；`create-hyperv-paths-and-vm.ps1` 是之後才加進去的第三支，補上了原本缺少的路徑分離與權限設定，是一個從「能動」演進到「路徑／權限都顧到」的典型小工具成長軌跡。

## 可延伸應用角度

- 適合做成「三步驟建置一台可用的 Hyper-V 測試 Host」教學影片，照著三支腳本的順序示範一次從裝機到開出第一台 VM。
- 可以跟先前收錄的 SUSE 測試環境準備工具放在同一個「測試環境自動化」系列裡，一個對應 Linux 端（SUSE 重灌後的收尾流程），一個對應 Windows 端（從裝 Hyper-V 角色到能跑 VM），做成平台對照集。
- `icacls` 遞迴授權搭配 `Set-VMHost` 那段，很適合單獨拆出來做一支短片，專講「Windows Server 把 VM／VHD 路徑搬離系統碟之後，為什麼會遇到存取被拒」這個常見雷區。
