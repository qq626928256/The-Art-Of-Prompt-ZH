<!--
name: 'Data: Agent SDK reference — Python'
description: Python Agent SDK reference including installation, quick start, custom tools via MCP, and hooks
ccVersion: 2.1.73
-->
# Agent SDK — Python

Claude Agent SDK 提供了一个高级接口,用于构建具有内置工具、安全功能和代理能力的 AI 代理。

## 安装

\`\`\`bash
pip install claude-agent-sdk
\`\`\`

---

## 快速开始

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="解释这个代码库",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

---

## 内置工具

| 工具        | 描述                          |
| ----------- | ----------------------------- |
| Read        | 读取工作区中的文件            |
| Write       | 创建新文件                    |
| Edit        | 对现有文件进行精确编辑        |
| Bash        | 执行 shell 命令               |
| Glob        | 按模式查找文件                |
| Grep        | 按内容搜索文件                |
| WebSearch   | 搜索网络信息                  |
| WebFetch         | 获取和分析网页                |
| AskUserQuestion | 向用户询问澄清问题            |
| Agent            | 生成子代理                    |

---

## 主要接口

### \`query()\` — 简单的一次性使用

\`query()\` 函数是运行代理的最简单方式。它返回一个异步的消息迭代器。

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="解释这个代码库",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

### \`ClaudeSDKClient\` — 完全控制

\`ClaudeSDKClient\` 提供对代理生命周期的完全控制。当你需要自定义工具、hooks、流式传输或中断执行的能力时使用它。

\`\`\`python
import anyio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AssistantMessage, TextBlock

async def main():
    options = ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    async with ClaudeSDKClient(options=options) as client:
        await client.query("解释这个代码库")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
\`\`\`

\`ClaudeSDKClient\` 支持:

- **上下文管理器** (\`async with\`) 用于自动资源清理
- **\`client.query(prompt)\`** 向代理发送提示
- **\`receive_response()\`** 流式传输消息直到完成
- **\`interrupt()\`** 在任务中途停止代理执行
- **自定义工具所必需** (通过 SDK MCP 服务器)

---

## 权限系统

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="重构认证模块",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Write"],
        permission_mode="acceptEdits"  # 自动接受文件编辑
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

权限模式:

- \`"default"\`: 对危险操作提示
- \`"plan"\`: 仅规划,不执行
- \`"acceptEdits"\`: 自动接受文件编辑
- \`"bypassPermissions"\`: 跳过所有提示(谨慎使用)

---

## MCP (模型上下文协议) 支持

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="打开 example.com 并描述你看到的内容",
    options=ClaudeAgentOptions(
        mcp_servers={
            "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

---

## Hooks

使用回调函数通过 hooks 自定义代理行为:

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get('tool_input', {}).get('file_path', 'unknown')
    print(f"已修改: {file_path}")
    return {}

async for message in query(
    prompt="重构 utils.py",
    options=ClaudeAgentOptions(
        permission_mode="acceptEdits",
        hooks={
            "PostToolUse": [HookMatcher(matcher="Edit|Write", hooks=[log_file_change])]
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

工具生命周期事件的 hook 回调输入(\`PreToolUse\`, \`PostToolUse\`, \`PostToolUseFailure\`)包括 \`agent_id\` 和 \`agent_type\` 字段,允许 hooks 识别是哪个代理(主代理或子代理)触发了工具调用。

可用的 hook 事件: \`PreToolUse\`, \`PostToolUse\`, \`PostToolUseFailure\`, \`UserPromptSubmit\`, \`Stop\`, \`SubagentStop\`, \`PreCompact\`, \`Notification\`, \`SubagentStart\`, \`PermissionRequest\`

---

## 常用选项

\`query()\` 接受一个顶级的 \`prompt\`(字符串)和一个 \`options\` 对象(\`ClaudeAgentOptions\`):

\`\`\`python
async for message in query(prompt="...", options=ClaudeAgentOptions(...)):
\`\`\`

| 选项                                | 类型    | 描述                                                              |
| ----------------------------------- | ------- | ----------------------------------------------------------------- |
| \`cwd\`                               | string  | 文件操作的工作目录                                                |
| \`allowed_tools\`                     | list    | 代理可以使用的工具 (例如 \`["Read", "Edit", "Bash"]\`)             |
| \`tools\`                             | list    | 可用的内置工具(限制默认集)                                        |
| \`disallowed_tools\`                  | list    | 明确禁用的工具                                                    |
| \`permission_mode\`                   | string  | 如何处理权限提示                                                  |
| \`mcp_servers\`                       | dict    | 要连接的 MCP 服务器                                               |
| \`hooks\`                             | dict    | 用于自定义行为的 hooks                                            |
| \`system_prompt\`                     | string  | 自定义系统提示                                                    |
| \`max_turns\`                         | int     | 停止前的最大代理轮数                                              |
| \`max_budget_usd\`                    | float   | 查询的最大预算(美元)                                              |
| \`model\`                             | string  | 模型 ID(默认:由 CLI 确定)                                        |
| \`agents\`                            | dict    | 子代理定义(\`dict[str, AgentDefinition]\`)                         |
| \`output_format\`                     | dict    | 结构化输出 schema                                                 |
| \`thinking\`                          | dict    | 思考/推理控制                                                     |
| \`betas\`                             | list    | 要启用的 Beta 功能(例如 \`["context-1m-2025-08-07"]\`)            |
| \`setting_sources\`                   | list    | 要加载的设置(例如 \`["project"]\`)。默认:无(无 CLAUDE.md 文件)    |
| \`env\`                               | dict    | 为会话设置的环境变量                                              |

---

## 消息类型

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage

async for message in query(
    prompt="查找 TODO 注释",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)
        print(f"停止原因: {message.stop_reason}")  # 例如 "end_turn", "max_turns"
    elif isinstance(message, SystemMessage) and message.subtype == "init":
        session_id = message.data.get("session_id")  # 捕获以供稍后恢复
\`\`\`

对于处理子代理任务事件时的更好类型安全性,提供了类型化任务消息子类:
- \`TaskStartedMessage\` — 在子代理任务注册时发出
- \`TaskProgressMessage\` — 带有累积使用指标的实时进度更新
- \`TaskNotificationMessage\` — 任务完成通知

---

## 子代理

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

async for message in query(
    prompt="使用 code-reviewer 代理审查此代码库",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep", "Agent"],
        agents={
            "code-reviewer": AgentDefinition(
                description="专家级代码审查员,用于质量和安全审查。",
                prompt="分析代码质量并提出改进建议。",
                tools=["Read", "Glob", "Grep"]
            )
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
\`\`\`

---

## 错误处理

\`\`\`python
from claude_agent_sdk import query, ClaudeAgentOptions, CLINotFoundError, CLIConnectionError, ResultMessage

try:
    async for message in query(
        prompt="...",
        options=ClaudeAgentOptions(allowed_tools=["Read"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)
except CLINotFoundError:
    print("未找到 Claude Code CLI。使用以下命令安装: pip install claude-agent-sdk")
except CLIConnectionError as e:
    print(f"连接错误: {e}")
\`\`\`

---

## 会话历史

使用顶级函数检索过去的会话数据:

\`\`\`python
from claude_agent_sdk import list_sessions, get_session_messages

# 列出所有过去的会话(同步函数 — 无需 await)
sessions = list_sessions()
for session in sessions:
    print(f"{session.session_id}: {session.cwd}")

# 获取特定会话的消息(同步函数 — 无需 await)
messages = get_session_messages(session_id="...")
for msg in messages:
    print(msg)
\`\`\`

---

## MCP 服务器管理

使用 \`ClaudeSDKClient\` 在运行时管理 MCP 服务器:

\`\`\`python
async with ClaudeSDKClient(options=options) as client:
    # 重新连接已断开的 MCP 服务器
    await client.reconnect_mcp_server("my-server")

    # 切换 MCP 服务器开/关
    await client.toggle_mcp_server("my-server", enabled=False)

    # 获取所有 MCP 服务器的状态
    status = await client.get_mcp_status()  # 返回 McpStatusResponse
\`\`\`

---

## 最佳实践

1. **始终指定 allowed_tools** — 明确列出代理可以使用的工具
2. **设置工作目录** — 始终为文件操作指定 \`cwd\`
3. **使用适当的权限模式** — 从 \`"default"\` 开始,仅在需要时升级
4. **处理所有消息类型** — 检查 \`ResultMessage\` 以获取代理输出
5. **限制 max_turns** — 使用合理的限制防止代理失控
