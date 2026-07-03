# Harness 快速診斷（2026-07-03，由 Fable 5 制度建立 session 撰寫）

本檔是整套制度的依據。後面每份規則檔都在修這裡列出的三個問題。
環境事實（撰寫當下實測與官方文件查證，非印象）：

- Repo：`jadewellhuang/examples`（pytorch/examples fork），撰寫前**沒有** CLAUDE.md、沒有專案級 `.claude/`。
- 執行環境：Claude Code 遠端容器（Claude Code on the web）。容器是暫時的，沒 commit+push 的東西 session 結束就消失。
- Agent tool 的 `model` 參數可接受值：`sonnet`、`opus`、`haiku`、`fable`。`fable` 只有特殊 session 才有，**規則檔一律不得依賴 fable**。
- 自訂 agent（`.claude/agents/*.md`）frontmatter 支援 `model` 與 `effort`（`low`/`medium`/`high`/`xhigh`/`max`），遠端 session 也會載入（來源：code.claude.com/docs/en/sub-agents.md、claude-code-on-the-web.md）。
- 內建 agent 類型：`general-purpose`、`Explore`（唯讀搜尋）、`Plan`（唯讀規劃）、`claude-code-guide`（查 Claude Code/API 文件）、`claude`。
- GitHub 操作只能走 `mcp__github__*` MCP 工具（沒有 `gh` CLI）。
- 大量工具是 deferred（schema 未載入），要先用 `ToolSearch` 以 `select:<名稱>` 載入才能呼叫。
- CLAUDE.md 的 `@path` import 是 session 啟動時**全文載入**，所以只 import 小檔，大檔用「純文字路徑＋觸發條件」引用。

## 第 1 名（最漏 token）：主對話親自下場做粗活

**症狀**：主對話直接 Read 大檔、Grep 掃整個 repo、用 GitHub MCP 拉整份 PR diff 或 CI log。這些原始輸出全部灌進主 context，一次 CI log 可以吃掉幾萬 token，而主對話真正需要的只是一句結論。

**具體修法**（詳見 `.claude/rules/delegation.md`）：
1. 凡是「讀很多、回報很少」的工作——掃 repo、大量讀檔、查網頁、讀 CI log、批次改檔——一律派 subagent，主對話只收結論與 `檔案:行號`。
2. GitHub MCP 呼叫一律帶 `minimal_output: true`（該工具支援時）、分頁 5–10 筆。CI 失敗 log 絕不直接拉進主對話，派 `haiku` subagent 讀完回報「哪個 job、哪一步、關鍵錯誤訊息三行內」。
3. 主對話允許親自讀的上限：單檔且已知確切路徑、預估 200 行以內。超過就派工。

## 第 2 名（最容易失焦）：零制度記憶，每個 session 從零重推

**症狀**：沒有 CLAUDE.md 之前，每個 session 都重新摸索 repo 結構、重新犯同樣的錯、重新做過的決定被推翻。長對話裡舊 context 被摘要壓縮後，模型會忘記最初的驗收條件，開始漂移（例如修 bug 修到一半跑去重構）。

**具體修法**：
1. 本套制度檔（CLAUDE.md 路由 + `.claude/rules/`）。**注意：這些檔案要 merge 進 `master` 之後，未來 session 才會自動載入。**
2. 每次踩坑把教訓寫回 `.claude/memory/lessons.md`（格式見 `.claude/rules/maintenance.md`），下個 session 不再踩。
3. 抗漂移錨點：接到多步驟任務時，先用一段話寫下「目標＋驗收條件」（用 TaskCreate 或寫進 scratchpad 檔案），每完成一步對照一次。發現自己在做驗收條件以外的事，停下來回到清單。

## 第 3 名（最容易出錯）：自驗自證，寫完就宣告完成

**症狀**：模型改完 code 憑「看起來對」就宣告完成。本 repo 特別危險：範例程式需要下載資料集、訓練要跑很久，弱模型會以「跑不動」為由完全跳過執行驗證，交出根本沒 import 成功的程式碼。

**具體修法**（驗證階梯，由便宜到貴，逐級做到做不動為止）：
1. **語法層**：改過的每個 `.py` 都跑 `python -m py_compile <檔案>`。零成本，不可跳過。
2. **執行層**：用最小參數實跑，例如 `python main.py --epochs 1 --no-cuda --batch-size 4`（本 fork 是舊版 pytorch/examples，**沒有** `--dry-run` 參數；先看該範例的 `add_argument` 有什麼可用）。注意：容器**沒有預裝 torch**，先跑 `pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu`（CPU 版較小）。裝不起來、或資料集下載失敗，算「環境受限」，要如實回報，不算驗證通過。
3. **驗收層**：完成宣告前，派一個 fresh-context subagent 做驗收（規則見 `.claude/rules/delegation.md` 的「驗證不自驗」）。改檔用 read-back，程式碼用實跑或測試。
4. 回報時區分三種狀態：「已驗證通過」「因環境限制只驗到第 N 層」「未驗證」。禁止把後兩者說成第一種。

## 誠實條款：這套制度補不了的

- 拆解、驗證、多樣本評審能把**執行品質**拉到接近本 session 的水準；但**模糊需求的詮釋**與**品味判斷**（API 設計取捨、文案語感、架構方向）補不了。遇到時的處置寫在 `.claude/rules/judgment.md` 的「品味判斷」節：能升級模型就升級（opus），不能就明說「這題超出我可靠判斷範圍」並列出選項讓使用者選，**不要假裝有把握**。
- 制度本身會退化。退化模式與預防寫在 `.claude/rules/letter.md`。
