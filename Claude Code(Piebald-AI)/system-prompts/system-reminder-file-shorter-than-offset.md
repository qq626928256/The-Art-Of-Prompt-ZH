<!--
name: 'System Reminder: File shorter than offset'
description: Warning when file read offset exceeds file length
ccVersion: 2.1.18
variables:
  - RESULT_OBJECT
-->
<system-reminder>警告：文件存在但比提供的偏移量（${RESULT_OBJECT.file.startLine}）要短。该文件有 ${RESULT_OBJECT.file.totalLines} 行。</system-reminder>

