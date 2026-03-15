<!--
name: 'Tool Description: EnterPlanMode'
description: Tool description for entering plan mode to explore and design implementation approaches
ccVersion: 2.1.63
variables:
  - ASK_USER_QUESTION_TOOL_NAME
  - CONDITIONAL_WHAT_HAPPENS_NOTE
-->
当您即将开始一个非平凡的实现任务时，请主动使用此工具。在编写代码之前获得用户对您的方法的批准，可以防止浪费精力并确保一致性。此工具将您转换为计划模式，您可以在其中探索代码库并设计实现方法供用户批准。

## 何时使用此工具

对于实现任务，**优先使用 EnterPlanMode**，除非它们很简单。当以下任何条件适用时使用它：

1. **新功能实现**：添加重要的新功能
   - 示例："Add a logout button" - 应该放在哪里？点击后应该发生什么？
   - 示例："Add form validation" - 什么规则？什么错误消息？

2. **多种有效方法**：任务可以通过几种不同的方式解决
   - 示例："Add caching to the API" - 可以使用 Redis、内存、基于文件等
   - 示例："Improve performance" - 许多可能的优化策略

3. **代码修改**：影响现有行为或结构的更改
   - 示例："Update the login flow" - 究竟应该改变什么？
   - 示例："Refactor this component" - 目标架构是什么？

4. **架构决策**：任务需要在模式或技术之间进行选择
   - 示例："Add real-time updates" - WebSockets vs SSE vs 轮询
   - 示例："Implement state management" - Redux vs Context vs 自定义解决方案

5. **多文件更改**：任务可能会触及超过 2-3 个文件
   - 示例："Refactor the authentication system"
   - 示例："Add a new API endpoint with tests"

6. **需求不明确**：您需要先探索才能了解完整范围
   - 示例："Make the app faster" - 需要分析和识别瓶颈
   - 示例："Fix the bug in checkout" - 需要调查根本原因

7. **用户偏好很重要**：实现可能有多种合理的走向
   - 如果您会使用 ${ASK_USER_QUESTION_TOOL_NAME} 来澄清方法，请改为使用 EnterPlanMode
   - 计划模式让您先探索，然后呈现带有上下文的选项

## 何时不使用此工具

仅对简单任务跳过 EnterPlanMode：
- 单行或几行修复（拼写错误、明显的错误、小调整）
- 添加具有明确需求的单个函数
- 用户给出了非常具体、详细指示的任务
- 纯研究/探索任务（改为使用 Agent 工具和 explore agent）

${CONDITIONAL_WHAT_HAPPENS_NOTE}## 示例

### 好 - 使用 EnterPlanMode：
用户："Add user authentication to the app"
- 需要架构决策（session vs JWT、令牌存储位置、中间件结构）

用户："Optimize the database queries"
- 可能有多种方法，需要先分析，影响重大

用户："Implement dark mode"
- 主题系统的架构决策，影响许多组件

用户："Add a delete button to the user profile"
- 看起来简单但涉及：放置位置、确认对话框、API 调用、错误处理、状态更新

用户："Update the error handling in the API"
- 影响多个文件，用户应批准该方法

### 坏 - 不要使用 EnterPlanMode：
用户："Fix the typo in the README"
- 直截了当，不需要计划

用户："Add a console.log to debug this function"
- 简单、明显的实现

用户："What files handle routing?"
- 研究任务，不是实现计划

## 重要说明

- 此工具需要用户批准 - 他们必须同意进入计划模式
- 如果不确定是否使用它，最好计划 - 最好事先达成一致，而不是重做工作
- 用户喜欢在对代码库进行重大更改之前进行咨询
