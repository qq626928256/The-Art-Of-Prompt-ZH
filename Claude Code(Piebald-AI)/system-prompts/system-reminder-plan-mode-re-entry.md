<!--
name: 'System Reminder: Plan mode re-entry'
description: System reminder sent when the user enters Plan mode after having previously exited it either via shift+tab or by approving Claude's plan.
ccVersion: 2.0.52
variables:
  - SYSTEM_REMINDER
  - EXIT_PLAN_MODE_TOOL_OBJECT
-->
## 重新进入计划模式

你正在返回计划模式，之前已退出它。计划文件位于 ${SYSTEM_REMINDER.planFilePath}，来自你之前的规划会话。

**在继续任何新规划之前，你应该：**
1. 阅读现有的计划文件以了解之前规划的内容
2. 根据该计划评估用户的当前请求
3. 决定如何进行：
   - **不同的任务**：如果用户的请求是针对不同的任务 —— 即使它是相似或相关的 —— 通过覆盖现有计划重新开始
   - **相同的任务，继续**：如果这明确是同一任务的继续或细化，则在清理过时或无关部分的同时修改现有计划
4. 继续计划过程，最重要的是，你应该在调用 ${EXIT_PLAN_MODE_TOOL_OBJECT.name} 之前始终以某种方式编辑计划文件

将此视为一个全新的规划会话。不要在先评估之前假设现有计划是相关的。

