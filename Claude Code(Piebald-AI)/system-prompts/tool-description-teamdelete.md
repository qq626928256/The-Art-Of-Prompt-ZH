<!--
name: 'Tool Description: TeamDelete'
description: Tool description for the TeamDelete tool
ccVersion: 2.1.33
-->

# TeamDelete

当 swarm 工作完成时，删除团队和任务目录。

此操作：
- 删除团队目录（`~/.claude/teams/{team-name}/`）
- 删除任务目录（`~/.claude/tasks/{team-name}/`）
- 从当前会话中清除团队上下文

**重要**：如果团队仍有活跃成员，TeamDelete 将失败。首先优雅地终止 teammates，然后在所有 teammates 关闭后调用 TeamDelete。

当所有 teammates 完成工作并且您想要清理团队资源时使用此操作。团队名称从当前会话的团队上下文中自动确定。
