<!--
name: 'Tool Description: TaskCreate'
description: Tool description for TaskCreate tool
ccVersion: 2.1.69
variables:
  - CONDTIONAL_TEAMMATES_NOTE
  - CONDITIONAL_TASK_NOTES
-->
使用此工具为当前编码会话创建结构化的任务列表。这有助于您跟踪进度、组织复杂任务，并向用户展示工作完成度。
它还有助于用户了解任务的进度和其请求的整体进度。

## 何时使用此工具

在以下场景中主动使用此工具：

- 复杂的多步骤任务 - 当任务需要 3 个或更多不同的步骤或操作时
- 非平凡且复杂的任务 - 需要仔细规划或多个操作的任务${CONDTIONAL_TEAMMATES_NOTE}
- 计划模式 - 使用计划模式时，创建任务列表来跟踪工作
- 用户明确要求 todo 列表 - 当用户直接要求您使用 todo 列表时
- 用户提供多个任务 - 当用户提供需要完成的事项列表（编号或逗号分隔）时
- 收到新指令后 - 立即将用户需求记录为任务
- 开始处理任务时 - 在开始工作之前将其标记为 in_progress
- 完成任务后 - 将其标记为已完成，并添加在实施过程中发现的任何新的后续任务

## 何时不使用此工具

在以下情况下跳过使用此工具：
- 只有一个简单直接的任务
- 任务很琐碎，跟踪它不会带来任何组织上的好处
- 任务可以在少于 3 个简单步骤内完成
- 任务纯粹是对话性或信息性的

请注意，如果只有一个琐碎任务要完成，您不应该使用此工具。在这种情况下，最好直接完成任务。

## 任务字段

- **subject**：简短、可操作的标题，采用祈使形式（例如，"Fix authentication bug in login flow"）
- **description**：详细描述需要完成的内容，包括上下文和验收标准
- **activeForm**（可选）：当任务处于 in_progress 状态时，在加载动画中显示的现在分词形式（例如，"Fixing authentication bug"）。如果省略，加载动画将显示 subject。

所有任务创建时状态均为 `pending`。

## 提示

- 创建具有清晰、具体 subject 的任务，描述结果
- 在 description 中包含足够的细节，以便其他代理能够理解并完成任务
- 创建任务后，如有需要，使用 TaskUpdate 设置依赖关系（blocks/blockedBy）
${CONDITIONAL_TASK_NOTES}- 首先检查 TaskList 以避免创建重复任务
