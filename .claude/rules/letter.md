# 給未來 session 的信（letter.md）

寫於 2026-07-03，制度建立 session（Fable 5）。讀者是你——未來在這個 repo 工作的模型。這封信講三件沒人問但最重要的事、這套制度會怎麼壞掉、以及交接狀態。

## 三件最重要的事

### 1. 沒 merge 進 master 的制度等於不存在，沒 push 的工作等於沒做

未來 session 只會載入 `master` 上的 CLAUDE.md 與 `.claude/`。制度更新若只停在分支，下個 session 根本看不到。所以：(a) 制度檔的改動要提醒使用者 merge；(b) 一般任務也一樣，容器 session 結束即回收，**commit+push 是任務的一部分，不是收尾選配**。做完一個可交付的小單位就 push 一次，不要攢。

### 2. 制度的量要配得上任務的量（比例原則）

這個 repo 是一堆互相獨立的小範例，日常任務多半很小。**三行的修改不需要四個 subagent。**規則的地板是：誠實回報三態、改過的 .py 跑 py_compile、做完 push——這三件事再小的任務都要做。至於派工的觸發條件在 delegation.md 第 1 節、verifier 的微改動例外在同檔第 6 節，**沒觸發就不要演**。制度是拿來省 token 和防錯的；如果你發現自己為了走流程花的 token 比做事還多，那是在錯用制度（也請把該案例記進 lessons.md）。

### 3. 你最大的風險不是能力不夠，是過度自信與過度順從

弱模型（其實所有模型）最常見的失敗不是解不出來，而是：把「應該可以」說成「已驗證」；使用者一質疑就立刻改口（不管原本對不對）；測試不過就想改測試。這套制度裡所有看起來囉嗦的規定——三態回報、verifier 不知情驗收、改考卷禁令——都是在擋這三件事。使用者質疑你的時候，正確反應是**重新查證據**，不是道歉換答案：證據支持你就有禮貌地堅持並秀出證據，證據不支持你就承認並記 lesson。

## 這套制度最可能的退化方式與預防

1. **儀式化空轉**：照樣派 verifier，但 prompt 裡先告訴它「我改好了，應該都對」→ 驗收變蓋章。預防：delegation.md 第 6 節明文禁止；你派驗收時自查 prompt 裡有沒有洩漏你的結論。
2. **規則通膨**：每次踩坑加一條，兩個月後沒人讀得完 → 全部失效。預防：maintenance.md 第 3 節的行數上限與精簡義務是硬規則，精簡跟新增一樣是貢獻。
3. **硬規則被「這次特殊」侵蝕**：嫌兩輪重試上限煩就先試第五次再說。預防：放寬硬規則必須先問使用者（maintenance.md 第 1 節）；「這次特殊」出現第三次就是規則要正式修訂的訊號，走流程改，不要默默不遵守。
4. **環境事實過期**：模型代號、工具參數、proxy 行為都會變。預防：規則檔裡的事實都帶日期；實測不符先派 `claude-code-guide` 查證再修（maintenance.md 第 4 節）。**不要因為一條事實過期就整份規則不信**。

## 建立時的假設（當時沒問使用者，若發現不對請修正）

- 假設使用者能接受制度檔放在 repo 裡隨 fork 公開可見。
- 假設規則檔用繁體中文寫（使用者以繁中溝通；工具名與參數保留英文）。
- 假設未來 session 的 Agent tool 至少有 `haiku`/`sonnet`/`opus` 三檔可選；若某代號失效，照 maintenance.md 第 4 節修事實，選擇邏輯（便宜→預設→升級三層）不變。
- 假設本 repo 之後的工作型態仍以「小範例的修改與問答」為主；若開始出現大型多檔開發，delegation.md 的閾值（200 行／3 檔）可能要調，先問使用者。

## 環境的坑（建立 session 實測）

- MCP server（github 等）會在 session 中途斷線又重連；工具突然消失先等一下或用 ToolSearch 重載，不要立刻回報「做不到」。
- Write/Edit 工具偶發 `Tool permission stream closed` 錯誤：原樣重試一次即可，通常會過。
- 容器沒有 torch；pip 裝 CPU 版可行但大，非必要別裝。範例實跑常卡在資料集下載（走 proxy）。
- Bash 裡 cat/grep/find 會被 hook 擋或提示改用專用工具，直接用 Read/Grep/Glob。

## 交接狀態（2026-07-03）

交付 A–G 全部完成並已 push 至分支 `claude/fable5-system-design-4atqjz`：diagnosis / CLAUDE.md / delegation（含 verifier、heavy-reasoner 兩個自訂 agent）/ judgment / templates / maintenance / 本信。收尾的對抗審查已跑過並修正（結果見 git log）。**等使用者 merge 進 master 制度才生效**。無未完成項目。
