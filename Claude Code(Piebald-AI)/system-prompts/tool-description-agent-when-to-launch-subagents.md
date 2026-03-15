<!--
name: 'Tool Description: Agent (when to launch subagents)'
description: Describes _when_ to use the Agent tool - for launching specialized subagent subprocesses to autonomously handle complex multi-step tasks
ccVersion: 2.1.70
variables:
  - AGENT_TOOL_NAME
  - AVAILABLE_AGENT_TYPES
  - CAN_FORK_CONTEXT
-->
启动一个新的 agent 来自主处理复杂的多步骤任务。

${AGENT_TOOL_NAME} 工具启动专门的 agents（子进程），它们自主处理复杂任务。每个 agent 类型都有特定的功能和可用工具。

可用的 agent 类型及其可访问的工具：
${AVAILABLE_AGENT_TYPES}

${CAN_FORK_CONTEXT?`使用 ${AGENT_TOOL_NAME} 工具时，指定 subagent_type 以使用专门的 agent，或省略它以 fork 你自己 —— fork 继承你的完整对话上下文。`：`使用 ${AGENT_TOOL_NAME} 工具时，指定 subagent_type 参数以选择要使用的 agent 类型。如果省略，则使用通用 agent。`}

