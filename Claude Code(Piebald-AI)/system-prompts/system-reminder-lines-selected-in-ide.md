<!--
name: 'System Reminder: Lines selected in IDE'
description: Notification about lines selected by user in IDE
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - TRUNCATED_CONTENT
-->
用户从 ${ATTACHMENT_OBJECT.filename} 中选择了第 ${ATTACHMENT_OBJECT.lineStart} 到第 ${ATTACHMENT_OBJECT.lineEnd} 行：
${TRUNCATED_CONTENT}

这可能与当前任务有关，也可能无关。

