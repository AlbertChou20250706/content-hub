OTP（One-Time Programmable）Flash Procedure 是 Fujitsu RX4770M8 系列主機板在量產（mass production）階段，對 BMC（AST2600）與 PRoT（AST1060，Platform Root of Trust）晶片燒錄一次性安全熔絲的完整作業程序：透過 TFTP 傳送 OTP image、在 BMC 的 AST 模式與 PRoT 的 uart console 下達燒錄指令，並用一份逐位元對照表驗證燒錄結果，藉此在硬體層級永久啟用 Secure Boot 與關閉除錯後門。

## 為什麼值得關注

OTP 熔絲燒錄是不可逆的操作——一旦寫入就無法復原，這在硬體驗證與量產流程裡是風險最高、最不能出錯的一步。這份素材把原本只存在資深工程師腦中的「燒錄前該檢查什麼、燒完該比對哪些暫存器值」整理成一份可重現的 SOP：三種機型（30/40/4770）各自的 `otp info strap` / `otp info conf` 預期輸出全部列出，任何人照著比對就能判斷這次燒錄是否正確，不需要燒錄的人自己去記那一長串十六進位暫存器代表什麼意思。對做硬體安全驗證或韌體量產把關的人來說，這種「如何把不可逆操作寫成有驗證步驟的 SOP」本身就是值得參考的方法論。

## 技術重點

實際下載並讀過 `M8_OTP_Flash_Step(30+40+47).txt` 與 `README_OTP.txt`（該 OTP image 的完整燒錄與驗證紀錄）之後，重點如下：

- **兩段式燒錄，燒兩顆不同晶片**：Step1 在 BMC（AST2600）的 uboot AST 模式下用 `tftp` 把 image 抓進記憶體、`otp prog 83000000` 燒錄；AC 斷電重開後，Step3 換到 PRoT（AST1060）的 uart console，下 `otp pb strap 2 1`、`otp pb strap 11 1`、`otp pb conf 0 F 1` 三道指令分別燒錄 strap 與 conf 區。兩顆晶片、兩套指令集、中間隔一次斷電重開，順序錯了就等於白燒。
- **OTP image 命名藏著版本語意**：README 說明 `OTPCFG5` 的 0xYYMMDDxX 編碼規則，最後一碼字母（A~F）分別對應 M5/M7 的 pre-production 或 mass production、以及 Secure Boot 是否啟用；這支 image（代號 D）對應「M7 量產版以後（含 AMDM2、1sktM6、M8），Secure Boot 功能已啟用」，代表這是正式對外出貨機的安全設定，不是內部除錯用的寬鬆版本。
- **OTP data region 內嵌 8 組 RSA-3072 公鑰**：README 完整列出 Key[0]~Key[7] 的 RSA modulus（各 384 bytes）與 exponent，型別是「OEM DSS public keys in Mode 2」，用於 Secure Boot 簽章驗證；Key[8]~Key[15] 保留未用。這些公鑰一旦燒進 OTP 就寫死在硬體裡，後續韌體必須用對應私鑰簽章才能通過開機驗證。
- **conf/strap 區直接鎖死除錯與備援路徑**：OTP configuration region 設定 Secure Boot Mode 2、RSA3072、SHA256、關閉 patch code、關閉 UART 開機；strap region 更直接把 `Disable ARM CM3`、`Disable ARM JTAG debug`、`Disable ARM JTAG trust world debug`、`Disable debug interfaces 0/1`、`Disable watchdog to reset full chip` 等除錯/繞過管道全部鎖死，且大部分 bit 都標記 `Write Protect`——這是「出貨後不能再被 debug 後門打開」的具體實作，不是紙上文件。
- **驗證靠逐位元比對，不是靠感覺**：`M8_OTP_Flash_Step` 提供 30/40/4770 三種機型各自完整的 `otp info strap`／`otp info conf` 預期輸出（每個 bit 的十六進位值與說明），燒錄完直接拿實際輸出逐行比對即可判斷成功與否，且文件明確標註「務必確保 MB 燈號熄滅後再上電」這種現場才會踩到的細節。
- **另附實體操作圖與正式流程 PDF**：資料夾內還有一張 `RX4770M8 OTP flash jumper location.png`（燒錄前要短接的實體 jumper 位置圖）與 `2way4wayM8_OTP_flash_procedure_v3.pdf`（雙路／四路 M8 機型的正式圖文版流程），是給產線或驗證人員對照實體主機板操作用的。

## 可延伸應用角度

- 適合做成「不可逆硬體操作的 SOP 設計」案例：對比一般軟體流程可以重跑，OTP 燒錄一次定生死，示範怎麼用「逐位元預期值比對表」把風險操作變成可驗證、可交接的標準流程。
- 可以搭配既有的 iRMC/BMC 相關素材（例如 iRMC_S6_NTP SOP）或安全主題，組成一集「伺服器 Secure Boot 是怎麼從韌體簽章一路鎖到硬體熔絲」的系列，把 RSA 公鑰、Secure Boot Mode、JTAG lockdown 這些抽象安全機制對應回實際燒錄指令。
- OTP image 命名規則裡藏的版本語意（pre-production vs mass production、Secure Boot 開關）本身可以獨立拉一段出來，講「怎麼從一個檔名就能判斷這份韌體的安全等級與適用階段」。
