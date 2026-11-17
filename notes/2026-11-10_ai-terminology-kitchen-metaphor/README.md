# AI 專有名詞對照：Skill／Project／Agent／MCP

> 筆記性質，不是要做成影片的主題。用「米其林廚房」比喻理解 Claude Code 的幾個核心概念，並對照我自己實際用到哪些。

## 廚房比喻

| 概念 | 比喻 | 說明 |
|---|---|---|
| Skill（技能包） | 食譜 | 需要時拿出來用，用完收回去，不佔多餘記憶體、不浪費 token。完整的技能包包含：提示詞模板、範例、輸出模板、程式調用，少一塊品質就會大幅下降 |
| Project（專案） | 整個廚房的配置 | 每次開啟都要把所有背景資訊重新載入，很吃上下文和記憶體。能用 Skill 解決的事，就不要重開 Project |
| Agent（智能助手） | 會判斷的廚師 | 不只是照食譜做，會判斷狀況、決定呼叫哪個 Skill 或工具。可分：主 Agent（獨立思考，負責複雜判斷）、Sub-Agent（執行機械性子任務）、管家 Agent（調度整個系統、分派任務省 token）。**對外輸出建議交給主 Agent**，Sub-Agent 思考力較弱，品質通常不如預期 |
| MCP | 倉庫跟冰箱 | 別人已經做好的工具／資源，直接拿來用，不用自己重建 |

## 對照我自己實際用到哪些

- **MCP（已經在用，而且用很多）**：這次跟 Claude Code 合作處理 `daily-tool-digest`、`content-hub` 相關工作時，查 GitHub Actions 紀錄、觸發 workflow、讀寫檔案，全部都是透過 MCP 工具（`mcp__github__*`、`mcp__Claude_Code_Remote__*` 等）完成——直接用 GitHub 平台已經建好的介面，不用自己寫一套串接程式。
- **Agent（已經有，而且不只一個）**：`daily-tool-digest`、`yt-daily-report`、`yt-auto-publish` 這幾個自動化，都是用 `claude -p` 搭配一份 PROMPT.md，在 GitHub Actions 裡全自動、無人值守運作——要自己判斷候選排除、README 缺失時暫緩還是繼續、今天是否已經跑過。這就是 Agent：會判斷狀況，不是死板照腳本。另外，Claude Code session 本身對我來說就是「主 Agent」，負責除錯、改 workflow、追蹤進度這類需要判斷的工作。
- **Skill（間接碰過，不是自己建的）**：Claude Code 處理我的事情時，背景會視情況載入各種 Skill，但這些是 Claude Code 自己在用的工具包，不是我為自己的自動化特別包裝建立的 Skill。目前 `daily-tool-digest` 等自動化是整份 PROMPT.md 直接丟給 Claude Code 讀，還沒走到拆成獨立 Skill、需要才調用的階段。
- **Project（目前沒用到）**：指的是 claude.ai 網頁版的 Project 功能（跟 Claude Code 不同）。目前的工作流程都是透過 Claude Code session + GitHub repo 進行，沒有用到這個。

## 紀錄

- 日期：2026-11-10
- 維護：Albert.Chou
