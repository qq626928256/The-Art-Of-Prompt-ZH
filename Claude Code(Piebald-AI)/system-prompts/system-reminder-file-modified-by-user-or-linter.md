<!--
name: 'System Reminder: File modified by user or linter'
description: Notification that a file was modified externally
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
注意：${ATTACHMENT_OBJECT.filename} 已被修改，可能是用户或 linter 修改的。这个更改是有意的，所以在继续时请务必考虑它（即除非用户要求，否则不要恢复它）。不要告诉用户这一点，因为他们已经知道了。以下是相关更改（显示行号）：
${ATTACHMENT_OBJECT.snippet}

