# WithMaxHDDFunctionTest：驗證「插滿硬碟」極限測試條件的自動化腳本組

`WithMaxHDDFunctionTest` 是一組針對伺服器「Function test with max. HDD」（插滿最大顆數硬碟）測試場景設計的驗證腳本，核心是 `Verify_MaxHDD_Conditions_v1.0.sh`：從硬碟顆數比對、RAID 陣列確認、Patrol Read 手動觸發、Idle 與負載下的功耗差異監看，到 dmesg／BMC SEL 穩定性檢查，一次跑完並輸出 HTML 報告。專案裡還搭了第二支工具 `Verify_SaveLogTool_Report_v1.0.sh`，用途是反過來驗證另一支自製工具 SaveLogTool 產出的成果物是否完整一致。

## 為什麼值得關注

「插滿最大顆數硬碟」是儲存子系統驗證的經典場景，但要把它做成有證據力的測試，痛點不在「知道要測什麼」，而在於瑣碎又容易漏掉的細節：OS 視角（`lsblk`）跟控制器視角（storcli/megacli/arcconf）看到的硬碟數常常對不上該用哪個當準；Patrol Read 從 Auto 切到 Manual 再手動觸發時，該怎麼證明它真的有跑、而且有造成功耗上升；OS 裝在單一顆 HDD 還是獨立 LUN 這種會影響測試結果解讀的細節，一般人工測試很容易忘記留證據。這支腳本把上述檢查點收斂成幾個 CLI flag，跑完直接留下文字 log 與 HTML 報告，兩份都是可以直接附給客戶或存檔稽核的格式。

## 技術重點（實際讀過原始碼）

- **雙視角硬碟計數，任一達標即 PASS**：`hdd_count_os()` 用 `lsblk -d -o TYPE | grep -c '^disk$'` 算 OS 看到的顆數，`hdd_count_ctrl()` 依 `--raid` 參數（storcli/megacli/arcconf/mdadm/skip）分派到對應工具的硬碟列表指令；只要兩個視角任一個 `>= --expected-hdds`（預設 12）就判定 PASS，避免因單一工具讀取失敗而誤判整體測試結果。
- **證據保存而非強行判定**：`root_src()` 只用 `findmnt -no SOURCE /` 記錄 OS 根目錄裝置，並在報告裡明確註記「不同控制器/LVM 可能無法 100% 自動判斷；人審更準確」——刻意不自動下 PASS/FAIL 結論，把「OS 是否裝在單一硬碟上」這種需要人工複核的項目留給人看，而不是硬做一個可能誤判的自動化判斷。
- **Patrol Read 全流程含功耗前後比對**：`patrol_manual()`/`patrol_start()` 依 RAID 工具送出對應指令（如 storcli 的 `set patrolread=off` 再 `start patrolread`），流程上會先讓系統 Idle 兩小時（`--idle-min`，可調），期間每 `--idle-min`/`POLL_SEC`（預設 30 秒）用 `ipmitool sensor` 抽樣功耗，記錄 idle 期間最低值當基準；觸發 Patrol Read 後再監看 10 分鐘，若功耗超過 idle 最低值 +10W 就判 PASS（代表 Patrol Read 確實在跑造成負載），否則標 WARN 讓人複查。
- **穩定性檢查覆蓋核心與 BMC 兩層**：一邊用 `dmesg -T | grep -iE 'panic|BUG:|Call Trace|power failure|resetting link'` 抓核心層異常，一邊用 `ipmitool sel elist` 過濾 `Power Unit|Failure|Fault` 關鍵字抓 BMC SEL 紀錄，兩邊都是 0 才算整體 PASS，任一邊有異常就標記需人工檢視 log。
- **純 Bash 產生互動式 HTML 報告**：沒有用任何模板引擎，靠 `row_kv()`/`row_pwr()` 兩個函式把資料以 `<script>row('kv', ...)</script>` 的形式逐行 append 到報告檔尾端，開啟 HTML 時再由內嵌 JS 的 `row()` 函式即時把資料插入對應表格 DOM——是一種很輕量、不依賴外部套件就能做出動態表格報告的寫法。
- **配套的部署與驗證腳本**：`makefolder.sh` 把兩支主腳本裝進 `/opt/maxhdd_test` 並設好權限；`SampleW.sh`/`ManualPatrol.sh` 是帶好範例 BMC IP／帳密的呼叫範本；`Packages.sh` 列出 `ipmitool`/`lsscsi`/`pciutils`/`usbutils`/`ethtool`/`dmidecode` 等前置套件安裝指令。另一支 `Verify_SaveLogTool_Report_v1.0.sh` 則是「驗證工具的工具」：檢查 SaveLogTool 產出的 manifest/csv/html/zip 是否齊全、用 `unzip -t` 驗證 ZIP 完整性、比對 CSV 列數與 ZIP 內檔案數是否一致，還能對 CSV 記錄的 SHA256 抽樣（預設 5 筆）跟 ZIP 內實際檔案的雜湊值做比對，且會自動找出 `--root` 目錄下最新一筆 manifest（不用手動指定路徑）。

## 可延伸應用角度

- 「Idle 基準值 → 觸發負載 → 比對功耗差異是否符合預期」這個模式，跟先前收錄的 [C-State 待機功耗記錄工具組]（2026-09-08）互補，可以做一集「功耗類驗證腳本比較」，對比兩者一個測「多低算正常待機」、一個測「觸發特定動作後功耗該漲多少才算真的有在跑」。
- `root_src()` 刻意不自動判定、只留證據給人審的設計，很適合拿來討論「自動化測試的邊界在哪裡」——不是所有判斷都該讓腳本下結論，尤其是 LVM/控制器組合多變的情境下，這是個具體反例可以講。
- `Verify_SaveLogTool_Report_v1.0.sh` 是「用一支工具驗證另一支工具產出」的 meta 驗證概念，可以單獨拉出來做一集「工程師怎麼確保自己寫的自動化工具沒有偷懶漏資料」，搭配 SHA256 抽樣比對的實際示範。
