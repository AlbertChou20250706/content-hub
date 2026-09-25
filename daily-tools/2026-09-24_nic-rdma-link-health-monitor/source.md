# nic_watch.sh（Albert Style）— 純終端機的 NIC / RDMA 連線健康度即時監看器

nic_watch.sh 是一支單檔 Bash 腳本，在純文字終端機上持續刷新顯示網卡（特別是綁定 RDMA 的網卡）連線健康度，同時列出每個計數器的「累計值」與「自上次刷新以來的增量」，錯誤一開始發生就能立刻看到數字在跳動，而不用等到累計值大到肉眼可辨。腳本只輸出到終端機，不寫任何檔案。

## 為什麼值得關注

RDMA／RoCEv2 網路調校最麻煩的地方在於：CRC 錯誤、封包丟棄這類問題往往是「偶發、緩慢累積」的，`ethtool -S` 單次執行只能看到一個累計總數，長時間跑下來的大數字很難判斷「現在到底有沒有在惡化」。這支腳本把問題簡化成一個判斷：只要 Delta 欄不是 0，就代表現在正在發生。它不需要安裝監控 agent、不用起背景服務，`./nic_watch.sh` 一行就能在 SSH 連線裡邊調線材/光模組邊看數字變化，剛好卡在「臨場排錯」與「正式監控系統」之間的空隙——比 `watch ethtool -S` 更聰明的地方在於它自動做了增量計算與顏色分級，比架一套 Prometheus/Grafana 更輕量、免設定。

## 技術重點

- **介面自動偵測有優先順序**：先掃 `/sys/class/infiniband/*/device/net` 找綁定 RDMA 裝置的網卡，找不到才退而求其次選「狀態 up 且速率最高」的介面（排除 `lo`）。這個順序反映了它的預設場景就是 RDMA/RoCEv2 環境，而非泛用的網路監控工具。
- **計數器用子字串比對，不是精確比對**：`WATCH_PATTERNS` 列了 `rx_errors`、`tx_errors`、`rx_crc_errors`、`rx_missed_errors`、`port.rx_discards` 等十幾種名稱，用 `case "${name}" in *"${p}"*)` 這種子字串匹配方式去對 `ethtool -S` 的輸出，因為不同驅動（Intel/Mellanox/Broadcom）對同一種錯誤的計數器命名不一致，子字串比對能跨驅動抓到而不用為每家硬體寫規則。其中 `rx_crc_errors`、`rx_errors`、`tx_errors`、`rx_dropped`、`tx_dropped`、`discards` 被標記為「致命」（`CRITICAL_PATTERNS`），增量時直接標紅並累計警示次數。
- **吞吐量是自己用 rx/tx bytes 差分算出來的**：沒有呼叫任何額外工具，單純讀 `/sys/class/net/<iface>/statistics/rx_bytes`／`tx_bytes` 兩次刷新間的差值，除以經過秒數再乘 8 除以 10^9 換算成 Gb/s——第一次刷新沒有基準值，所以畫面上會先顯示「measuring」。
- **MTU/RDMA active_mtu 的判斷值是寫死的 RDMA 調校經驗值**：MTU 非 9000 顯示黃色（RoCEv2 建議開 jumbo frame），`ibv_devinfo` 讀到的 `active_mtu` 非 4096 也顯示黃色（MTU 9000 對應的 InfiniBand active_mtu 理論值就是 4096），這兩個數字組合是這支腳本專門針對 RDMA 網路調校場景寫死的判斷邏輯，不是泛用網路工具會內建的知識。
- **輸出到非終端機時自動關閉顏色碼**：`[ ! -t 1 ]` 判斷 stdout 是否為 tty，若被導向檔案或管線則把所有 `C_*` 變數清空，但腳本每次刷新仍會呼叫 `clear`，README 裡也明講「不適合直接導向記錄檔，要記錄請用別的腳本」——這是刻意只做「即時看」而不做「留存記錄」的取捨。

## 可延伸應用角度

適合跟先前收錄的 E800/E835 RDMA 驅動安裝工具（`daily-tools/2026-09-22_e800-e835-rdma-driver-setup-toolkit`）放在同一支「RDMA 環境搭建到監看」系列裡講——一個負責把驅動裝起來、驗證 RDMA 是否啟用，這個負責裝完之後盯著連線品質看有沒有掉包。也可以單獨做一支「終端機監控腳本設計」主題影片，示範它如何只用 `/sys` 和 `ethtool` 兩個資料源、不依賴任何常駐服務，就做出接近商用監控工具的「累計 vs 增量」呈現方式，適合中小型 lab 或臨場除錯場景參考。
