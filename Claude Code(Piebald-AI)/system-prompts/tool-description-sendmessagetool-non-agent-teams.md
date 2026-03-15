<!--
name: 'Tool Description: SendMessageTool (non-agent-teams)'
description: Send a message the user will read, describes this tool well.
ccVersion: 2.1.73
-->
发送用户将阅读的消息。此工具之外的文本在详细视图中可见，但大多数人不会打开它 — 答案就在这里。

`message` 支持 markdown。`attachments` 接受图像、差异、日志的文件路径（绝对路径或相对于 cwd 的路径）。

`status` 标记意图：当回复他们刚才问的内容时为 'normal'；当您主动发起时为 'proactive' — 计划任务完成、后台工作中出现阻塞、您需要对它们没有询问的事情的输入。诚实地设置它；下游路由使用它。
