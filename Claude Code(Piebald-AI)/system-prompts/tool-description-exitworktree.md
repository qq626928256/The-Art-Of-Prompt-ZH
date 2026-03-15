<!--
name: 'Tool Description: ExitWorktree'
description: Roughly, the reverse of the ExitWorktree
ccVersion: 2.1.72
-->
退出由 EnterWorktree 创建的 worktree 会话，并将会话返回到原始工作目录。

## 范围

此工具仅对在此会话中由 EnterWorktree 创建的 worktree 进行操作。它不会触及：
- 您使用 `git worktree add` 手动创建的 worktree
- 来自先前会话的 worktree（即使当时是由 EnterWorktree 创建的）
- 如果从未调用过 EnterWorktree，您所在的目录

如果在 EnterWorktree 会话之外调用，该工具是 **no-op**：它报告没有活动的 worktree 会话，并且不采取任何操作。文件系统状态保持不变。

## 何时使用

- 用户明确要求"退出 worktree"、"离开 worktree"、"回去"或以其他方式结束 worktree 会话
- 不要主动调用此工具 — 仅当用户要求时

## 参数

- `action`（必需）：`"keep"` 或 `"remove"`
  - `"keep"` — 在磁盘上保留 worktree 目录和分支。如果用户想稍后回来工作，或者有要保留的更改，请使用此选项。
  - `"remove"` — 删除 worktree 目录及其分支。当工作完成或放弃时，使用此选项进行干净退出。
- `discard_changes`（可选，默认为 false）：仅对 `action: "remove"` 有意义。如果 worktree 有未提交的文件或不在原始分支上的提交，除非将其设置为 `true`，否则工具将拒绝删除它。如果工具返回列出更改的错误，在用 `discard_changes: true` 重新调用之前与用户确认。

## 行为

- 将会话的工作目录恢复到 EnterWorktree 之前的位置
- 清除 CWD 依赖的缓存（系统提示部分、内存文件、计划目录），以便会话状态反映原始目录
- 如果 tmux 会话附加到 worktree：在 `remove` 时终止，在 `keep` 时保持运行（返回其名称以便用户可以重新连接）
- 退出后，可以再次调用 EnterWorktree 来创建新的 worktree
