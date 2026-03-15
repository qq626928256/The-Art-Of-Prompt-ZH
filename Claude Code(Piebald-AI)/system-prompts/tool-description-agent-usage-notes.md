<!--
name: 'Tool Description: Agent (usage notes)'
description: Usage notes and instructions for the Task/Agent tool, including guidance on launching subagents, background execution, resumption, and worktree isolation
ccVersion: 2.1.70
variables:
  - TOOL_BASE_DESCRIPTION
  - TOOL_PARAMETERS_DESCRIPTION
  - GET_TIER_FN
  - IS_TRUTHY_FN
  - PROCESS_OBJECT
  - IS_SUBAGENT_CONTEXT_FN
  - HAS_SUBAGENT_TYPES
  - TOOL_OBJECT
  - IS_TEAMMATE_CONTEXT_FN
  - ADDITIONAL_USAGE_NOTES
  - EXTRA_USAGE_NOTES
  - SUBAGENT_TYPE_DEFINITIONS
  - DEFAULT_AGENT_DESCRIPTION
-->
${TOOL_BASE_DESCRIPTION}
${TOOL_PARAMETERS_DESCRIPTION}

使用说明：
- 始终包含一个简短的描述（3-5 个词），总结 agent 将要做的工作${GET_TIER_FN()!=="pro"?`
- 尽可能并发启动多个 agent，以最大化性能；为此，请使用包含多个工具调用的单条消息`:""}
- 当 agent 完成后，它将向您返回一条消息。agent 返回的结果对用户不可见。要向用户显示结果，您应该向用户发送一条文本消息，简要总结结果。${!IS_TRUTHY_FN(PROCESS_OBJECT.env.CLAUDE_CODE_DISABLE_BACKGROUND_TASKS)&&!IS_SUBAGENT_CONTEXT_FN()&&!HAS_SUBAGENT_TYPES?`
- 您可以选择使用 run_in_background 参数在后台运行 agent。当 agent 在后台运行时，它完成时会自动通知您 — 不要 sleep、轮询或主动检查其进度。继续其他工作或回应用户。
- **前台与后台**：当您需要 agent 的结果才能继续时，请使用前台（默认）— 例如，研究结果 agent 的发现将指导您的后续步骤。当您确实有独立的并行工作时，请使用后台。`:""}
- 可以使用 `resume` 参数通过传递先前调用的 agent ID 来恢复 agent。恢复后，agent 将继续保留其完整的先前上下文。${HAS_SUBAGENT_TYPES?"当不恢复并且您指定 subagent_type 时，每次调用都是新的开始，您应该提供详细的任务描述和所有必要的上下文。":"当不恢复时，每次调用都是新的开始，您应该提供详细的任务描述和所有必要的上下文。"}
- 当 agent 完成后，它将向您返回一条消息及其 agent ID。如果需要后续工作，您可以使用此 ID 稍后恢复 agent。
${!HAS_SUBAGENT_TYPES?`- 提供清晰、详细的提示，以便 agent 可以自主工作并准确返回您需要的信息。
`:""}- agent 的输出通常应该被信任
- 明确告诉 agent 您是否期望它编写代码或只是进行研究（搜索、文件读取、web 获取等）${HAS_SUBAGENT_TYPES?"":",因为它不知道用户的意图"}
- 如果 agent 描述提到应该主动使用它，那么您应该尽力在用户要求之前使用它。请自行判断。
- 如果用户指定希望您"并行"运行 agent，您必须发送一条包含多个 ${TOOL_OBJECT.name} 工具使用内容块的消息。例如，如果您需要并行启动 build-validator agent 和 test-runner agent，请发送一条包含两个工具调用的消息。
- 您可以选择设置 `isolation: "worktree"` 在临时 git worktree 中运行 agent，为其提供存储库的隔离副本。如果 agent 没有进行任何更改，worktree 将自动清理；如果进行了更改，worktree 路径和分支将在结果中返回。${IS_SUBAGENT_CONTEXT_FN()?`
- run_in_background、name、team_name 和 mode 参数在此上下文中不可用。仅支持同步子 agent。`:IS_TEAMMATE_CONTEXT_FN()?`
- name、team_name 和 mode 参数在此上下文中不可用 — teammates 不能生成其他 teammates。省略它们以生成子 agent。`:""}${ADDITIONAL_USAGE_NOTES}${EXTRA_USAGE_NOTES}

${HAS_SUBAGENT_TYPES?SUBAGENT_TYPE_DEFINITIONS:DEFAULT_AGENT_DESCRIPTION}
