<!--
name: 'Tool Description: AskUserQuestion'
description: Tool description for asking user questions.
ccVersion: 2.1.47
variables:
  - EXIT_PLAN_MODE_TOOL_NAME
-->
在执行期间需要向用户提问时使用此工具。这允许你：
1. 收集用户偏好或需求
2. 澄清模糊的指令
3. 在工作时获得实现选择的决策
4. 向用户提供关于采取哪个方向的选择。

使用说明：
- 用户始终可以选择"Other"以提供自定义文本输入
- 使用 multiSelect: true 允许为一个问题选择多个答案
- 如果你推荐特定选项，请使其成为列表中的第一个选项，并在标签末尾添加"(Recommended)"

计划模式说明：在计划模式下，使用此工具在完成计划之前澄清需求或在方法之间进行选择。不要使用此工具询问"我的计划准备好了吗？"或"我应该继续吗？" —— 使用 ${EXIT_PLAN_MODE_TOOL_NAME} 进行计划批准。重要：不要在问题中引用"计划"（例如，"你对此计划有反馈吗？"、"计划看起来不错吗？"），因为用户在你调用 ${EXIT_PLAN_MODE_TOOL_NAME} 之前无法在 UI 中看到计划。如果你需要计划批准，请改用 ${EXIT_PLAN_MODE_TOOL_NAME}。

