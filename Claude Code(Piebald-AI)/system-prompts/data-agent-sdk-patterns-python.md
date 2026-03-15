<!--
name: 'Data: Agent SDK patterns — Python'
description: Python Agent SDK patterns including custom tools, hooks, subagents, MCP integration, and session resumption
ccVersion: 2.1.73
-->
# Agent SDK 模式 — Python

## 基础 Agent

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Explain what this repository does",
        options=ClaudeAgentOptions(
            cwd="/path/to/project",
            allowed_tools=["Read", "Glob", "Grep"]
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

---

## 自定义工具

自定义工具需要 MCP 服务器。使用 \`ClaudeSDKClient\` 以获得完全控制（自定义 SDK MCP 工具需要 \`ClaudeSDKClient\` — \`query()\` 仅支持外部 stdio/http MCP 服务器）。

\`\`\`python
import anyio
from claude_agent_sdk import (
    tool,
    create_sdk_mcp_server,
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    TextBlock,
)

@tool("get_weather", "Get the current weather for a location", {"location": str})
async def get_weather(args):
    location = args["location"]
    return {"content": [{"type": "text", "text": f"The weather in {location} is sunny and 72°F."}]}

server = create_sdk_mcp_server("weather-tools", tools=[get_weather])

async def main():
    options = ClaudeAgentOptions(mcp_servers={"weather": server})
    async with ClaudeSDKClient(options=options) as client:
        await client.query("What's the weather in Paris?")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
\`\`\`

---

## Hooks

### 工具使用后 Hook

记录任何编辑后的文件更改：

\`\`\`python
import anyio
from datetime import datetime
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get('tool_input', {}).get('file_path', 'unknown')
    with open('./audit.log', 'a') as f:
        f.write(f"{datetime.now()}: modified {file_path}\n")
    return {}

async def main():
    async for message in query(
        prompt="Refactor utils.py to improve readability",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit", "Write"],
            permission_mode="acceptEdits",
            hooks={
                "PostToolUse": [HookMatcher(matcher="Edit|Write", hooks=[log_file_change])]
            }
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

---

## 子 Agent

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

async def main():
    async for message in query(
        prompt="Use the code-reviewer agent to review this codebase",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Glob", "Grep", "Agent"],
            agents={
                "code-reviewer": AgentDefinition(
                    description="Expert code reviewer for quality and security reviews.",
                    prompt="Analyze code quality and suggest improvements.",
                    tools=["Read", "Glob", "Grep"]
                )
            }
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

---

## MCP 服务器集成

### 浏览器自动化 (Playwright)

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Open example.com and describe what you see",
        options=ClaudeAgentOptions(
            mcp_servers={
                "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
            }
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

### 数据库访问 (PostgreSQL)

\`\`\`python
import os
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Show me the top 10 users by order count",
        options=ClaudeAgentOptions(
            mcp_servers={
                "postgres": {
                    "command": "npx",
                    "args": ["-y", "@modelcontextprotocol/server-postgres"],
                    "env": {"DATABASE_URL": os.environ["DATABASE_URL"]}
                }
            }
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

---

## 权限模式

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    # Default: prompt for dangerous operations
    async for message in query(
        prompt="Delete all test files",
        options=ClaudeAgentOptions(
            allowed_tools=["Bash"],
            permission_mode="default"  # Will prompt before deleting
        )
    ):
        pass

    # Plan: agent creates a plan before making changes
    async for message in query(
        prompt="Refactor the auth system",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit"],
            permission_mode="plan"
        )
    ):
        pass

    # Accept edits: auto-accept file edits
    async for message in query(
        prompt="Refactor this module",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit"],
            permission_mode="acceptEdits"
        )
    ):
        pass

    # Bypass: skip all prompts (use with caution)
    async for message in query(
        prompt="Set up the development environment",
        options=ClaudeAgentOptions(
            allowed_tools=["Bash", "Write"],
            permission_mode="bypassPermissions"
        )
    ):
        pass

anyio.run(main)
\`\`\`

---

## 错误恢复

\`\`\`python
import anyio
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    CLINotFoundError,
    CLIConnectionError,
    ProcessError,
    ResultMessage,
)

async def run_with_recovery():
    try:
        async for message in query(
            prompt="Fix the failing tests",
            options=ClaudeAgentOptions(
                allowed_tools=["Read", "Edit", "Bash"],
                max_turns=10
            )
        ):
            if isinstance(message, ResultMessage):
                print(message.result)
    except CLINotFoundError:
        print("Claude Code CLI not found. Install with: pip install claude-agent-sdk")
    except CLIConnectionError as e:
        print(f"Connection error: {e}")
    except ProcessError as e:
        print(f"Process error: {e}")

anyio.run(run_with_recovery)
\`\`\`

---

## 会话恢复

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage

async def main():
    session_id = None

    # First query: capture the session ID
    async for message in query(
        prompt="Read the authentication module",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Glob"])
    ):
        if isinstance(message, SystemMessage) and message.subtype == "init":
            session_id = message.data.get("session_id")

    # Resume with full context from the first query
    async for message in query(
        prompt="Now find all places that call it",  # "it" = auth module
        options=ClaudeAgentOptions(resume=session_id)
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`

---

## 会话历史

\`\`\`python
from claude_agent_sdk import list_sessions, get_session_messages

# List past sessions (sync function — no await)
sessions = list_sessions()
for session in sessions:
    print(f"Session {session.session_id} in {session.cwd}")

# Retrieve messages from the most recent session (sync function — no await)
if sessions:
    messages = get_session_messages(session_id=sessions[0].session_id)
    for msg in messages:
        print(msg)
\`\`\`

---

## 自定义系统提示

\`\`\`python
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Review this code",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Glob", "Grep"],
            system_prompt="""You are a senior code reviewer focused on:
1. Security vulnerabilities
2. Performance issues
3. Code maintainability

Always provide specific line numbers and suggestions for improvement."""
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
\`\`\`
