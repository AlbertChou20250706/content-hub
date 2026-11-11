Route_Status 是一支 Windows 用的路由／網卡／連通性檢查工具，專門診斷「Wi-Fi + 有線網路雙網卡共存」時最常見的陷阱：以為路徑對了，其實走錯網卡，或 ICMP 通但 TCP 服務根本連不上。

## 為什麼值得關注

README 裡記錄了一個 2026-11-03 的真實案例：Chrome 連 `apex.mic.com.tw:7001` 出現 `ERR_CONNECTION_TIMED_OUT`，用乙太網路測試時 ICMP 通（1ms）但 TCP 7001 失敗，換成 Wi-Fi 卻能連上——結論是實驗室網段（乙太網路）到公司系統的 TCP 被防火牆政策擋住，只有 Wi-Fi 網段被允許。這支工具存在的理由，就是把「ICMP 通 ≠ TCP 通」這個教訓，變成一支會自動比對出口網卡、Ping 和實際 TCP 埠三件事情的腳本，而不是每次都要工程師手動 `tracert`、`route print`、`Test-NetConnection` 拼湊判斷。README 裡還記了一個更隱蔽的坑：手動 `route add` 測試用的路由不會自動消失，殘留的 `/32` 主機路由會讓目標「看起來正常」，實際上是被覆蓋掉——這種誤判在排查網路問題時特別容易浪費時間。

## 技術重點

實際下載並讀過 README 與 835 行的 `Route_Status.ps1`（搭配 123 行的 `Route_Status.bat` 啟動器）後，幾個值得注意的設計：

- **把「設計檢查」寫進腳本邏輯**：透過 `$PreferredDefaultAlias`（預設 `Wi-Fi`）定義哪張網卡該是主要預設出口，腳本不只檢查路由存不存在，還會判斷目前的預設路由配置是否符合這個設計原則，不符合就 WARN 並直接附上 `Set-NetIPInterface` 修正指令。
- **分層檢查結構**：L1（網卡/預設路由/同網段/Metric）→ L2（ARP 重複 MAC）→ L3（名稱解析/出口網卡/期望路徑/ICMP/TCP/另一網卡比對），每一層對應「底層 Code & Logic → 中層 Data & Logging → 高層 Business & Context」，checks 本身被設計成可疊加而非互相獨立。
- **`-ProbeAltPath` 的安全設計**：TCP 失敗時可選擇新增暫時性 `/32` 主機路由改走另一張網卡再測一次，驗證「是不是只有特定網段被允許」，但這個操作被限定在獨立旗標、預設關閉，且一定在 `finally` 區塊移除，避免像 README 自己記錄的陷阱一樣遺留測試路由。
- **結束代碼設計成可被排程腳本消費**：0＝全部正常、10＝有 WARN、20＝有 FAIL/CRIT/ERROR，其他值（如 1）代表 PowerShell 本身執行失敗而非檢查結果——這讓這支工具可以被包進更大的自動化流程當健康檢查節點使用。
- **HTML 報告直接沿用 DMTF Redfish-Protocol-Validator 的版面規範**（頂部漸層列＋即時篩選、結果磁磚點擊篩選、可摺疊章節卡片），README 明確寫「之後新寫的腳本，HTML log 一律沿用這個版面」，代表這份報告模板正在變成 Albert 系列工具共用的視覺標準，而不是這支工具專屬的呈現方式。
- **README 的「給未來 AI Agents 的維護說明」章節**列出多個實測踩過的坑（`.bat` 不能放中日文否則 cmd 把命令列切碎、`.ps1` 必須是 UTF-8 with BOM、`param()` 必須是第一個可執行語句、位元位移要注意 uint32 溢位），這些細節比一般工具 README 更偏向「給下一個維護者/agent 的操作手冊」，而不只是使用說明。

## 可延伸應用角度

- 可以做成一支「IT 排查工具設計」系列短片，核心主題就是「ICMP 通 ≠ TCP 通」這個真實案例，示範怎麼把一次性排查的手動步驟，收斂成一支會自動判斷網卡角色與設計正確性的腳本。
- README 裡「給未來 AI Agents 的維護說明」與「編碼與語法規則」這兩個章節，本身就值得單獨拿出來做一支「寫 PowerShell/Batch 工具時容易踩的雷」short，尤其是中日文編碼、BOM、`param()` 順序這幾個已經實測過的真實錯誤。
- HTML 報告版面沿用 DMTF Redfish-Protocol-Validator 樣式這個決定，可以跟先前介紹過的 Redfish 相關工具串成「Albert style 報告模板」系列，講解同一套視覺規範怎麼套用到不同檢查腳本上。
