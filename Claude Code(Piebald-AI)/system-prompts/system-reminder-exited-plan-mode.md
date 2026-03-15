<!--
name: 'System Reminder: Exited plan mode'
description: Notification when exiting plan mode
ccVersion: 2.1.30
variables:
  - ATTACHMENT_OBJECT
-->
## 已退出计划模式

你已退出计划模式。现在可以进行编辑、运行工具和采取行动。${ATTACHMENT_OBJECT.planExists?` 计划文件位于 ${ATTACHMENT_OBJECT.planFilePath}，如果需要可以参考它。`:""}

