LAN Throughput Benchmark Suite 是 Albert.Chou 針對「網卡吞吐量驗證」場景寫的一組互動式測試腳本合集：一套用 `iperf3` 做傳統乙太網路 LAN Throughput 測試（`lan_throughput.sh`，對應 Ubuntu/RedHat/SUSE 三種發行版各自的版本線），另一套用 `perftest`（`ib_write_bw`）做 RDMA 網卡的頻寬驗證，分別為 Intel E835-CCQDA1M（`E835_RDMA_Bench.sh`）與 AMD POLLARA-1Q400P（`RDMA_LAN_Throughput_Test.sh`）各自量身打造。兩條線的腳本都遵循同一套「uni/bi 方向 × duration/sweep/stress 模式」互動選單設計，並用相同的三重日誌（終端彩色輸出＋純文字 log＋HTML 報告）與版本比對機制串起來。

## 為什麼值得關注

這批工具跟之前介紹過的 `E800_setup`（Intel E835 驅動安裝工具）是同一條 RDMA 驗證流水線的下一棒——驅動裝好、RoCEv2 切好之後，接下來要回答「這張卡到底能跑出多少頻寬、有沒有跑到滿速」，這正是這套 Benchmark Suite 在做的事。它的價值不只是「跑一個 iperf3/perftest 指令」，而是把 SIT 產線上真正會踩到的坑都寫進了腳本與 README：例如 iperf3 的 `Transfer`（GBytes）和 `Bitrate`（Gbits/sec）是兩個不同單位、容易被誤讀；`[SUM]` 才是全部 stream 加總、單一 stream 數字拿來當結論會低估真實頻寬；400G 網卡不開到 16~32 條並列流就會被單核 CPU 卡住上限。這些都是「工具能跑」跟「結果能信」之間的落差，也是這系列工具真正的技術含金量所在。

## 技術重點

實際下載並讀過 `Ubuntu24.4/For iperf3/README.md`（`lan_throughput.sh` v1.7.0）、`Ubuntu24.4/For RDMA/200GB/E835CCQDA1M/E835_RDMA_Bench/v1.9.0/README.md`、以及 `Ubuntu24.4/For RDMA/400GB/AMD POLLARA-1Q400P/v1.1.2/README.md` 之後，重點如下：

- **版本演進本身就是一份踩坑史**：`lan_throughput.sh` 的 Version History 從 v1.0.0 一路寫到 v1.7.0，每版都對應一個具體 bug——v1.4.0 修「Duration/Stress 模式緩衝到結束才顯示，長測試像當機」（改成即時串流輸出）；v1.6.0 修「結果解析誤抓單一 stream 數值，跟 `[SUM]` 對不上」；v1.7.0 修「UDP block size 超過 65507 bytes 會直接報錯」，改成自動下修。這種把每次修正原因寫進 CHANGELOG 的習慣，本身就是很好的「工具如何在生產環境中被打磨」的素材。
- **Ubuntu 24.04 原生化的具體做法**：腳本明確用 `ufw`（非 `firewalld`）、`ip` 指令（非 `ifconfig`），並在 System Prep 步驟自動處理防火牆關閉、IP/MTU 9000 設定、`sysctl kernel.numa_balancing=0`；針對高速網卡另外調大 `net.core.rmem_max`/`wmem_max` 等 socket buffer，直接解決大 `-w` 時常見的 `socket buffer size not set correctly` 報錯。
- **RDMA 端額外處理 PCIe 層級的效能陷阱**：AMD POLLARA 版本的 v1.1.2 新增「Disable ACS」——用 `setpci ECAP_ACS+0x6.w=0000` 清除所有 ACS-capable 裝置的 Access Control Services，因為 ACS 開啟時 PCIe P2P 流量會被強制繞經 root complex，增加延遲、壓低頻寬；同時搭配 `numactl` 做網卡對應的 NUMA 節點綁定。這兩項是很多人做 RDMA 效能測試時會漏掉、但直接影響跑分的系統層調校。
- **顏色與 log 格式的穩定性設計**：所有終端色碼改用 `\033`（ESC）而非 `\e`/`\x1b`，確保在 Serial Console 與舊版 SSH client 都能正確顯示；`[WARN]` 等級被明確定義成「非錯誤、可繼續」，避免長時間 Stress 測試中的提醒訊息被誤判成失敗。
- **實測數據具體到可比對**：README 附上真實跑分——Intel E835 200GbE 在 4 小時 soak 測試中 sweep peak 達 196.02 Gb/s 且零波動；AMD POLLARA-1Q400P 在 uni-duration 測得平均 390.15 Gb/sec。這些數字讓工具不只是「能跑」，而是有明確的效能基準可供後續版本比對回歸。

## 可延伸應用角度

- 可以跟 `E800_setup`（驅動安裝）接成兩集系列：第一集講「怎麼把 RDMA 網卡的驅動跟 RoCEv2 裝起來」，第二集講「裝完之後怎麼驗證頻寬有沒有跑滿、怎麼讀懂 iperf3/perftest 的輸出」，完整覆蓋一張網卡從上架到驗收的流程。
- 「Transfer vs Bitrate」「SUM vs per-stream」這兩個 iperf3 讀圖陷阱很適合單獨拉成一支短影片，用腳本 README 裡的實際輸出範例當教材，講給任何要看懂 iperf3 報告的人聽，不限定 SIT 背景的觀眾。
- ACS 停用對 RDMA P2P 頻寬的影響可以跟 NUMA 綁定放在同一集「PCIe 層級效能調校」裡，串連之前 NVQual 系列談過的 PCIe 相關工具，形成一條「PCIe 效能驗證」的小主題線。
