# NVQual USB Controller Blacklist Helper — 讓 GPU 品質認證測試不再無故重開機

> 來源路徑：`NVQual/USB_Controller_Blacklist/`（`nvqual_usb_blacklist.sh` v1.2.1）
> 設計者：Albert Chou　最後更新：2026-08-06

---

## 這是什麼

一支 4 階段（SCAN → WRITE → RECHECK → AUDIT）的 Bash 稽核工具，專門解決 NVIDIA GPU 伺服器品質認證測試（NVQual）在跑 Test #31（D3 Power Management）時會在約 7 分鐘處無故自行重開機、測試永遠跑不完的問題。它負責把伺服器上所有 USB controller 正確寫進 NVQual 的 blacklist 設定，並在寫入後重新從硬碟讀回驗證、再做一次完整稽核，確保「以為設定好了」和「實際上真的生效」之間沒有落差。

## 為什麼值得關注

這不是一支憑空寫的工具，而是一次真實踩坑後固化下來的解法，README 裡完整保留了排查歷程：

- **症狀**：Test #31 執行到第 7 分鐘左右，系統自己重開機，測試中斷，NVQual 無法完成驗證。
- **第一個坑（格式）**：NVQual 只認巢狀路徑 `custom_configuration.blacklist_device`；如果照直覺寫成平面的 `{"blacklist": [...]}`，NVQual 會「安靜地」完全忽略這個設定，畫面上看起來設定檔存在、內容也對，但實際上等於沒設，測試照樣重開機。這種「設定被默默吃掉、沒有任何錯誤訊息」的坑，是最容易讓人卡很久的類型。
- **第二個坑（少列）**：4 顆 USB controller 只要漏列 1 顆（尤其是最容易被忽略的主 controller `0000:09:00.0`），D3 電源循環測到那顆時一樣會連帶觸發重開機。
- **真正的環境根因**：透過 KVM over IP（BMC 虛擬 USB）遠端操作時，KVM 會在主 USB controller 上掛虛擬鍵鼠、虛擬光碟、虛擬磁碟（`AMI Virtual HDisk`）。D3 電源循環測試把這些虛擬裝置電源關掉再開啟時，虛擬裝置沒有回應，kernel log 灌爆 `cannot reset (err = -110)` / `timing out command, waited 60s`，最終拖垮 I/O 導致系統重開機。改在實體機（非遠端 KVM）直接執行，環境乾淨，問題就消失了。

這種「工具本身沒錯，但遠端操作環境引入了額外干擾，且設定格式錯誤又不會報錯」的組合，是伺服器測試 debug 裡很典型、卻很少被寫下來的一類問題。

## 技術重點（實際看過程式碼）

- **4 階段流程**：STEP1 SCAN（列舉全部 USB controller）→ STEP2 WRITE（選擇並寫入 blacklist）→ STEP3 RECHECK（寫入後重新讀回硬碟比對，不相信「寫了就等於生效」）→ STEP4 AUDIT（JSON 語法、路徑型別、BDF 格式、重複值、sysfs 實際存在性等多項稽核）。
- **雙來源 PCI 列舉＋交叉驗證**：預設用 `lspci -Dnnk -d ::0c03`，但也實作了純 sysfs（`/sys/bus/pci/devices`，class `0x0c03`）列舉當 fallback，兩者都在時會交叉比對數量（`crosscheck_sources`），避免單一工具本身出錯卻沒被發現。沒有 `lspci`（例如精簡版 Ubuntu Server）也能靠 sysfs 跑完全部流程。
- **共用的 dotted-path JSON schema 存取層**（`_schema.py`，由 shell 透過 heredoc 動態產生）：`get_list()` / `set_list()` / `detect_path()` 統一處理巢狀路徑讀寫，STEP2/3/4 都 import 同一份實作，避免同樣的邏輯寫三份、改一次要改三處。而且寫入時會先偵測既有設定檔已經用哪個路徑（`nvqual` 巢狀 / `flat` 平面 / 其他候選路徑），自動跟隨既有結構寫入，不會憑空多開一把 NVQual 讀不到的 key。
- **版本紀錄裡藏了一個真實 bug 修復**：v1.2.1 修掉了 `grep -c` 對空檔案回傳 `"0\n0"` 導致 `[ -eq ]` 判斷式壞掉的問題，統一改用 `wc -l` 並做數值消毒——這種「工具檢查工具自己的邊界情況」的細節，是判斷一支腳本是不是真的上過production的訊號。
- **配套的 `nvqual_grub_fix.sh`**：另一條可選 workaround，用真正的 key=value parser（而非粗暴字串替換）去合併 `GRUB_CMDLINE_LINUX_DEFAULT`，加上時間戳記備份、`--rollback`、`--dry-run`、`--remove`，具備冪等性。README 裡明確記錄：這條路徑能壓下部分錯誤，但**最終確認不是必要條件**，只要 blacklist 完整、且在實體機執行就夠——這段「試過但最後證明非必要」的誠實記錄，本身就是很好的教材。
- **輸出**：6 級式 log（INFO/PASS/WARN/FAIL/CRIT/ERROR）＋可摺疊 HTML report＋純文字 OS log，每次執行都存成帶時間戳的獨立資料夾（`usbbl_<timestamp>/{reports,logs,json,debug}`），方便留存證據。

## 可延伸應用角度

- **「設定檔格式錯誤但沒有任何報錯」的通用教訓**：可以單獨拉出來做一支「巢狀 vs 平面 JSON key 被靜默忽略」的短片，不限 NVQual，任何讀取巢狀設定但只做寬鬆解析的工具都可能踩到這種坑，是很適合做成通用軟體工程教學的素材。
- **KVM 虛擬裝置干擾實體測試**這個環境因素本身也值得獨立成一集：遠端管理介面（BMC/KVM）注入的虛擬 USB/CD/HDD 裝置，如何在電源管理類測試中製造「假故障」，這對做伺服器/硬體測試的觀眾是很實用但少見的知識點。
- NVQual 資料夾裡還有另外兩支同一情境下的姊妹工具可以一起做成小系列：`PCIe Link Training Prep & Diagnostic Tool`（NVQual Test #24–#31 前的環境淨化腳本，安全終止 DCGM/persistenced 並清乾淨 NVIDIA 驅動）與 `GPU_verifies_USB_capacity_compatibility`（GPU 對 USB 容量相容性驗證，含 Clonezilla SOP）。三支工具串起來就是一套完整的「NVIDIA GPU 伺服器品質認證前置準備到疑難排解」流程，適合做成 2–3 集的迷你系列。
- 可以搭配既有的 SOP 類內容（例如 `topics/2026-08-12_chouap-cloud-migration-sop`）的呈現風格，強調「真實踩坑紀錄 + 可重複套用的 SOP」這條路線，是這個頻道已經驗證有效的內容型態。
