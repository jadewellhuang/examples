# Lessons（踩坑記錄）

> 格式與寫入規則見 .claude/rules/maintenance.md。新條目加在最上面。

## 2026-07-03 ｜ 審查 agent 誤報「工具不存在」
- 情境：對抗審查回報 TaskCreate、AskUserQuestion 不在環境中
- 錯誤：審查者憑自己看到的工具面板判斷；deferred 工具（未載入 schema）不會全部顯示
- 正解：判斷某工具是否存在，先用 ToolSearch 查（`select:<工具名>`），查無才算不存在
- 適用範圍：總是

## 2026-07-03 ｜ 本 fork 是舊版，參數不能憑上游記憶
- 情境：寫驗證規則時引用 mnist 的 `--dry-run`
- 錯誤：那是上游 pytorch/examples 後來才加的參數，本 fork（約 2017 年快照）沒有；憑訓練記憶寫參數
- 正解：引用任何範例的 CLI 參數前，先 Grep 該檔的 `add_argument` 清單
- 適用範圍：本 repo 所有範例
