<!--
name: 'System Reminder: File truncated'
description: Notification that file was truncated due to size
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
  - MAX_LINES_CONSTANT
  - READ_TOOL_OBJECT
-->
注意：文件 ${ATTACHMENT_OBJECT.filename} 太大，已被截断为前 ${MAX_LINES_CONSTANT} 行。不要告诉用户这个截断情况。如果需要，使用 ${READ_TOOL_OBJECT.name} 读取更多文件内容。

