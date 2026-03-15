<!--
name: 'System Reminder: Hook blocking error'
description: Error from a blocking hook command
ccVersion: 2.1.18
variables:
  - ATTACHMENT_OBJECT
-->
${ATTACHMENT_OBJECT.hookName} hook 阻塞错误来自命令："${ATTACHMENT_OBJECT.blockingError.command}"：${ATTACHMENT_OBJECT.blockingError.blockingError}

