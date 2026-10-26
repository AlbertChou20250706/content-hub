Network Configuration Manager v2.0.0 是一支用 USB 隨身碟帶到目標主機上執行的互動式雙網卡設定腳本，透過文字選單讓使用者設定內網/外網介面、DHCP 或靜態 IP，並產生 CLI 日誌、HTML 報表、JSON 狀態檔三種紀錄。

## 為什麼值得關注

機房或 SIT 測試環境裡常常會遇到「這台機器還沒上網域、沒有既有網路設定，只能靠隨身碟帶腳本進去跑」的場景——尤其是雙網卡機器要分清楚哪個介面接內網（測試機房網段）、哪個接外網（公司網路或 DHCP），手動用 `ip`/`nmcli` 一個個設定容易漏步驟。這支工具把整套「主機端準備 USB → 被測端掛載執行 → 互動選單設定 → 產生報表」的流程寫成一支單一 bash 腳本，文件也很完整：README、QUICK_START、CHANGELOG 齊全，甚至畫了一張 USB 上傳流程的 SVG 圖。

但實際讀完 500 行腳本後發現一個值得拿出來講的落差：文件宣稱是「企業級雙網卡管理、自動容錯」，但核心的 `apply_configuration()` 函式其實沒有呼叫任何 `ip addr add`／`nmcli`／`netplan apply` 之類會真正改網路設定的指令，裡面只是針對已選的介面印出「Configuring ${INNER_INTERFACE}...」再 `sleep 1`、寫 log、印 ✓ 成功訊息——全程是模擬（simulation），並不會真的套用設定。連 QUICK_START 自己的「驗證配置」章節也要使用者自己手動跑 `ip addr show`、`ping` 去確認,而不是腳本驗證自己改的東西。這跟 README 裡「70 年經驗的首席設計師」這種顯然灌水的角色介紹放在一起看,很像是先把文件、選單、日誌格式這些「外殼」做滿,核心「真的改網路」的那一步還沒寫。

## 技術重點

- **真正做到的部分**：`detect_interfaces()` 用 `ip link show` + `ip addr show` + `ip link show | grep link/ether` 抓出每張介面的狀態、IP、MAC，存進 `INTERFACE_STATE` 這個 bash 關聯陣列,這段是會真的執行、有實際輸出的。
- **沒做到的部分**：`configure_inner_network()`/`configure_outer_network()` 只負責收集使用者輸入(IP、netmask、gateway 或 DHCP 選擇)並寫進變數與 log,`apply_configuration()` 完全沒有把這些輸入轉成系統指令——全程搜尋腳本找不到任何 `ip addr add`、`ip route`、`netplan`、`nmcli` 字樣。
- **三種輸出格式**：CLI log、HTML 報表(黑底設計)、JSON 狀態檔,三者都在 `finalize_logs()` 統一收尾寫入 `/root/Documents/`,JSON 格式設計得頗完整(metadata/interfaces/configuration/events 分層),如果之後要補上真正的套用邏輯,這層輸出結構是可以直接沿用的。
- **USB 工作流本身的設計**：資料夾裡附的 `MD/usb_script_upload_workflow.svg` 搭配 README 把「主機端格式化 FAT32→複製檔案→sync→被測端掛載執行」整套流程圖像化,這部分對缺乁操作手冊的團隊是實用的,跟腳本本身能不能真的改網路是兩件獨立的事。

## 可延伸應用角度

- 適合做成「讀文件 vs. 讀程式碼」對照案例:示範如何不被一份寫得很完整、看起來很有自信的 README 說服,而是直接去看核心函式有沒有真的呼叫會動系統的指令——這對正在學著評估別人(或 AI 生成)程式碼的工程師是很實用的習慣。
- 也可以跟同系列已收錄的「文件拖稿、功能超前」案例(OT Calculator,2026-09-04)對照,組成「文件與實作落差」的系列:一支是功能做好了但部署檔沒跟上,這支則是文件、UI、日誌殼都做滿了但核心邏輯還是空的——兩種很有代表性的個人工具開發階段。
- USB 隨身碟帶腳本上機、分主機端/被測端兩階段操作的這套流程,本身可以單獨抽出來做「無網路環境機台初始化」的教學素材,不受限於這支腳本現在还没写完的网络配置逻辑。
