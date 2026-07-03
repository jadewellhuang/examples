# 模型調度守則（delegation.md）

讀者：未來每個主 session 的模型（可能是 Sonnet 或更小）。所有規則以「照做就對」為標準寫成；例外情況照 `.claude/rules/judgment.md` 判斷。

## 1. 何時派工（指揮官不下場）

主對話的職責是拆解、派工、整合、對使用者回報。粗活派出去。

**必須派 subagent 的情境**（符合任一條就派）：
- 要讀的內容預估超過 400 行，或要碰超過 3 個檔案
- 掃 repo 找東西（不確定在哪個檔）→ 用 `Explore` agent
- 查網頁、查文件 → `general-purpose`（或 Claude Code 問題用 `claude-code-guide`）
- 讀 CI log、PR diff、任何「原始內容大、有用結論小」的東西
- 批次套用同一種修改到多個檔案

**不要派工的情境**（派工有冷啟動成本，每個 agent 要自己重建 context）：
- 已知確切路徑、單檔 200 行以內的閱讀
- 一兩個 Bash 指令能回答的事（如 `git log`、`ls`）
- 對已在 context 裡的內容做小修改

**並行**：互相獨立的子任務，在同一個回合一次派出（Agent tool 可並行呼叫），不要串行等。

## 2. 模型與 effort 選擇（按本環境實際可用值）

Agent tool 的 `model` 參數可用：`haiku`、`sonnet`、`opus`（`fable` 只在特殊 session 存在，**不得依賴**）。
effort **無法**在 Agent tool 呼叫時指定；只能透過自訂 agent 定義的 frontmatter 設定。本 repo 已提供兩個（`.claude/agents/`）：

| 派誰 | 用於 | 呼叫方式 |
|---|---|---|
| `haiku` | 機械性、低歧義：log 摘要、格式轉換、已定型 pattern 的批次套用、簡單彙整 | `subagent_type: "general-purpose"` 或 `"Explore"` + `model: "haiku"` |
| `sonnet`（預設工作馬） | 實作、重構、研究、審查、搜尋 | `subagent_type: "general-purpose"`（不帶 model 即繼承，或明寫 `model: "sonnet"`） |
| `verifier`（自訂，sonnet/medium） | 驗收：read-back、跑測試、對照驗收條件 | `subagent_type: "verifier"` |
| `heavy-reasoner`（自訂，opus/high） | 升級用：連錯後的疑難排查、架構取捨、第二意見 | `subagent_type: "heavy-reasoner"` |

判斷模糊時的預設：`sonnet`。省錢不是往下猜的理由——haiku 只給你**確定**是機械性的活。

## 3. 派工三件套（每個 Agent prompt 必含，缺一不派）

1. **目標與動機**：做什麼＋為什麼（動機讓 agent 遇到歧義時能自己做對的取捨）。
2. **驗收條件**：可勾選的清單，agent 完成前要逐條自查。
3. **回報格式**：明確規定回什麼、多長、什麼格式。

現成模板在 `.claude/rules/templates.md`，五種任務型態（搜尋／實作／重構／研究／審查）直接填空，不要自己即興寫。

另外必附：**已知背景**（相關檔案路徑、已排除的方向、相關的 lessons.md 條目）。subagent 看不到主對話，你不寫它就不知道。

## 4. 回報合約（寫進每個派工 prompt 的固定條款）

- 只回：結論、關鍵證據（`檔案:行號` 格式）、驗收條件逐條勾選結果、遇到的阻礙。
- 禁止把大段檔案內容、完整 log 貼回來。長產物寫成檔案（scratchpad 或 repo 內），回報路徑。
- 回報上限預設 30 行；研究型任務可放寬到 60 行，要在派工時明說。
- 失敗也要照格式回報：試了什麼、卡在哪、最後的錯誤訊息（≤5 行）。

## 5. 升降級路徑

- **haiku 錯一次** → 不重試，直接升 `sonnet` 重派（附 haiku 的失敗回報）。
- **sonnet 同一子任務連錯兩次** → 升 `heavy-reasoner`，prompt 必須附**完整失敗軌跡**：兩次分別改了什麼、驗證怎麼跑的、錯誤輸出各 ≤10 行。不附軌跡等於讓 opus 重犯一樣的錯。
- **降級回收**：heavy-reasoner 解出問題後，若同型問題還有多處，把「解法 pattern」寫成明確步驟，降回 `haiku`/`sonnet` 批次套用，不要讓 opus 做完全部。
- **重試上限**：同一個問題、同一種方法，全 session 最多兩輪。第三次之前必須三選一：(a) 換方法、(b) 升級模型、(c) 照 judgment.md 的判準問使用者。禁止原樣第三試。

## 6. 驗證不自驗

寫的人不能自己宣告通過。完成宣告前：

- **檔案／文件改動** → 派 `verifier` 做 read-back：重新讀檔，對照驗收條件逐條判定，回報每條 pass/fail。
- **程式碼改動** → 派 `verifier` 跑驗證階梯（見 `.claude/rules/diagnosis.md` 第 3 節）：`py_compile` → 最小參數實跑。verifier 只看檔案與執行結果，派工 prompt **不要**告訴它「我覺得已經做對了」。
- **高風險判斷**（不可逆操作前的決定、對外發布內容、架構方向）→ 二選一：
  - 第二意見：派 `heavy-reasoner` 獨立評估同一問題（給它原始問題，不給你的答案，比對結論）。
  - 多答案評審：派 2–3 個 agent 各自獨立解，再派一個 fresh agent 當評審選優。成本高，只用在錯了很難回頭的地方。
- verifier 回報 fail → 回到第 5 節的升降級路徑處理，不得「解釋掉」fail 直接宣告完成。
