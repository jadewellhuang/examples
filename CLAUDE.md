# CLAUDE.md — 路由與最低限度規則

這個 repo 是 pytorch/examples 的**舊版** fork：一堆彼此獨立的 PyTorch 範例（mnist、dcgan、vae、word_language_model…）。每個子目錄自成一體，沒有共用函式庫、沒有測試套件、沒有 CI（截至 2026-07）。改 A 目錄不影響 B 目錄。

## 每個 session 都適用的五條

1. **指揮官不下場**：掃 repo、大量讀檔、查網頁、讀 CI log、批次改檔 → 派 subagent，主對話只收結論與 `檔案:行號`。已知確切路徑、預估 200 行以內的單檔才可親自讀。何時派、選什麼 model/effort → 讀 `.claude/rules/delegation.md`。
2. **驗證階梯**：改過的每個 `.py` 必跑 `python -m py_compile <檔案>`；能實跑就用最小參數實跑（容器沒預裝 torch，見下方環境備忘）；完成宣告前派 fresh-context agent 驗收（微改動例外的定義見 `.claude/rules/delegation.md` 第 6 節）。回報必須區分「已驗證／因環境限制部分驗證／未驗證」，禁止混用。
3. **先寫驗收條件**：多步驟任務開工前，先寫下目標與驗收條件（用當時環境實有的任務清單工具如 TaskCreate/TodoWrite，或 scratchpad 檔案），每完成一步對照一次；發現自己在做清單外的事就停下來回到清單。
4. **踩坑回寫**：犯錯被使用者糾正、或發現環境的意外行為 → 當場把教訓寫進 `.claude/memory/lessons.md`（格式見 `.claude/rules/maintenance.md`），寫完再繼續原任務。
5. **不確定就查，查不到就標註**：不得編造 API、路徑、參數值。Claude Code 本身的問題派 `claude-code-guide` agent 查官方文件；查不到就寫「UNVERIFIED」。

## 按情境讀（純路徑引用，不是 @import，需要時才讀）

| 情境 | 讀這個檔 |
|---|---|
| 要派 subagent：何時派、model/effort 怎麼選、升降級、驗證不自驗 | `.claude/rules/delegation.md` |
| 派工時要現成 prompt 模板（搜尋／實作／重構／研究／審查） | `.claude/rules/templates.md` |
| 拿不準的判斷：要不要升級模型、算不算完成、該不該問使用者、方向是不是錯了 | `.claude/rules/judgment.md` |
| 要修改 `.claude/` 底下任何檔案（含本檔） | `.claude/rules/maintenance.md`，先讀完再動手 |
| 想了解這套制度的來歷、harness 弱點分析、已知退化風險 | `.claude/rules/diagnosis.md`、`.claude/rules/letter.md` |
| 過去踩過的坑 | `.claude/memory/lessons.md` |

## 環境備忘（2026-07 實測；發現過時就照 maintenance.md 修）

- 遠端容器（Claude Code on the web），session 結束即回收：**沒 commit+push 的都會消失**。
- 沒有 `gh` CLI，GitHub 一律走 `mcp__github__*` MCP 工具；呼叫帶 `minimal_output: true`（該工具支援時）、分頁 5–10 筆。
- 很多工具是 deferred：先用 `ToolSearch` 以 `select:<工具名>` 載入 schema 才能呼叫。
- 容器**沒有預裝 torch**。要實跑範例：`pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu`。裝不起來或資料集下載失敗（網路走 proxy）＝環境受限，如實回報。
- 本 fork 是舊版程式碼：mnist 等範例**沒有** `--dry-run`；最小化實跑用 `--epochs 1 --no-cuda --batch-size 4` 這類參數，先看該檔的 `add_argument` 清單。
- commit 訊息與 PR 內文不要提內部模型 ID。
