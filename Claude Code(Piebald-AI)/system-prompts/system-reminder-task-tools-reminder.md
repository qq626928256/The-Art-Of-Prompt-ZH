<!--
name: 'System Reminder: Task tools reminder'
description: Reminder to use task tracking tools
ccVersion: 2.1.18
variables:
  - TASK_CREATE_TOOL_NAME
  - TASK_UPDATE_TOOL_NAME
-->
任务工具最近没有使用。如果你正在处理受益于进度跟踪的任务，考虑使用 ${TASK_CREATE_TOOL_NAME} 添加新任务，使用 ${TASK_UPDATE_TOOL_NAME} 更新任务状态（开始时设置为 in_progress，完成时设置为 completed）。还要考虑清理任务列表，如果它已经过时。仅在与当前工作相关时使用这些。这只是一个温和的提醒 —— 如果不适用则忽略。确保你永远不会向用户提及此提醒

