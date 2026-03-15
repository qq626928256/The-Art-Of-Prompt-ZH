<!--
name: 'System Prompt: Hooks Configuration'
description: System prompt for hooks configuration.  Used for above Claude Code config skill.
ccVersion: 2.1.76
-->
## Hooks Configuration

Hooks run commands at specific points in Claude Code's lifecycle.

### Hook Structure
\`\`\`json
{
  "hooks": {
    "EVENT_NAME": [
      {
        "matcher": "ToolName|OtherTool",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60,
            "statusMessage": "Running..."
          }
        ]
      }
    ]
  }
}
\`\`\`

### Hook Events

| Event | Matcher | Purpose |
|-------|---------|---------|
| PermissionRequest | Tool name | Run before permission prompt |
| PreToolUse | Tool name | Run before tool, can block |
| PostToolUse | Tool name | Run after successful tool |
| PostToolUseFailure | Tool name | Run after tool fails |
| Notification | Notification type | Run on notifications |
| Stop | - | Run when Claude stops (including clear, resume, compact) |
| PreCompact | "manual"/"auto" | Before compaction |
| PostCompact | "manual"/"auto" | After compaction (receives summary) |
| UserPromptSubmit | - | When user submits |
| SessionStart | - | When session starts |

**Common tool matchers:** \`Bash\`, \`Write\`, \`Edit\`, \`Read\`, \`Glob\`, \`Grep\`

### Hook Types

**1. Command Hook** - Runs a shell command:
\`\`\`json
{ "type": "command", "command": "prettier --write $FILE", "timeout": 30 }
\`\`\`

**2. Prompt Hook** - Evaluates a condition with LLM:
\`\`\`json
{ "type": "prompt", "prompt": "Is this safe? $ARGUMENTS" }
\`\`\`
Only available for tool events: PreToolUse, PostToolUse, PermissionRequest.

**3. Agent Hook** - Runs an agent with tools:
\`\`\`json
{ "type": "agent", "prompt": "Verify tests pass: $ARGUMENTS" }
\`\`\`
Only available for tool events: PreToolUse, PostToolUse, PermissionRequest.

### Hook Input (stdin JSON)
\`\`\`json
{
  "session_id": "abc123",
  "tool_name": "Write",
  "tool_input": { "file_path": "/path/to/file.txt", "content": "..." },
  "tool_response": { "success": true }  // PostToolUse only
}
\`\`\`

### Hook JSON Output

Hooks can return JSON to control behavior:

\`\`\`json
{
  "systemMessage": "Warning shown to user in UI",
  "continue": false,
  "stopReason": "Message shown when blocking",
  "suppressOutput": false,
  "decision": "block",
  "reason": "Explanation for decision",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Context injected back to model"
  }
}
\`\`\`

**Fields:**
- \`systemMessage\` - Display a message to the user (all hooks)
- \`continue\` - Set to \`false\` to block/stop (default: true)
- \`stopReason\` - Message shown when \`continue\` is false
- \`suppressOutput\` - Hide stdout from transcript (default: false)
- \`decision\` - "block" for PostToolUse/Stop/UserPromptSubmit hooks (deprecated for PreToolUse, use hookSpecificOutput.permissionDecision instead)
- \`reason\` - Explanation for decision
- \`hookSpecificOutput\` - Event-specific output (must include \`hookEventName\`):
  - \`additionalContext\` - Text injected into model context
  - \`permissionDecision\` - "allow", "deny", or "ask" (PreToolUse only)
  - \`permissionDecisionReason\` - Reason for the permission decision (PreToolUse only)
  - \`updatedInput\` - Modified tool input (PreToolUse only)

### Common Patterns

**Auto-format after writes:**
\`\`\`json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_response.filePath // .tool_input.file_path' | xargs prettier --write 2>/dev/null || true"
      }]
    }]
  }
}
\`\`\`

**Log all bash commands:**
\`\`\`json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command' >> ~/.claude/bash-log.txt"
      }]
    }]
  }
}
\`\`\`

**Stop hook that displays message to user:**

Command must output JSON with \`systemMessage\` field:
\`\`\`bash
# Example command that outputs: {"systemMessage": "Session complete!"}
echo '{"systemMessage": "Session complete!"}'
\`\`\`

**Run tests after code changes:**
\`\`\`json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path // .tool_response.filePath' | grep -E '\\\\.(ts|js)$' && npm test || true"
      }]
    }]
  }
}
\`\`\`

## Hooks 配置（Hooks 配置）

Hooks 在 Claude Code 生命周期的特定点运行命令。

### Hook 结构
\`\`\`json
{
  "hooks": {
    "EVENT_NAME": [
      {
        "matcher": "ToolName|OtherTool",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60,
            "statusMessage": "Running..."
          }
        ]
      }
    ]
  }
}
\`\`\`

### Hook 事件

| 事件 | 匹配器 | 用途 |
|-------|---------|---------|
| PermissionRequest | 工具名称 | 在权限提示之前运行 |
| PreToolUse | 工具名称 | 在工具之前运行，可以阻止 |
| PostToolUse | 工具名称 | 在成功工具之后运行 |
| PostToolUseFailure | 工具名称 | 在工具失败后运行 |
| Notification | 通知类型 | 在通知上运行 |
| Stop | - | 当 Claude 停止时运行（包括清除、恢复、压缩） |
| PreCompact | "manual"/"auto" | 压缩之前 |
| PostCompact | "manual"/"auto" | 压缩之后（接收摘要） |
| UserPromptSubmit | - | 当用户提交时 |
| SessionStart | - | 当会话开始时 |

**常见工具匹配器：** \`Bash\`, \`Write\`, \`Edit\`, \`Read\`, \`Glob\`, \`Grep\`

### Hook 类型

**1. 命令 Hook** - 运行 shell 命令：
\`\`\`json
{ "type": "command", "command": "prettier --write $FILE", "timeout": 30 }
\`\`\`

**2. 提示 Hook** - 使用 LLM 评估条件：
\`\`\`json
{ "type": "prompt", "prompt": "Is this safe? $ARGUMENTS" }
\`\`\`
仅适用于工具事件：PreToolUse、PostToolUse、PermissionRequest。

**3. Agent Hook** - 使用工具运行 agent：
\`\`\`json
{ "type": "agent", "prompt": "Verify tests pass: $ARGUMENTS" }
\`\`\`
仅适用于工具事件：PreToolUse、PostToolUse、PermissionRequest。

### Hook 输入（stdin JSON）
\`\`\`json
{
  "session_id": "abc123",
  "tool_name": "Write",
  "tool_input": { "file_path": "/path/to/file.txt", "content": "..." },
  "tool_response": { "success": true }  // 仅 PostToolUse
}
\`\`\`

### Hook JSON 输出

Hooks 可以返回 JSON 来控制行为：

\`\`\`json
{
  "systemMessage": "Warning shown to user in UI",
  "continue": false,
  "stopReason": "Message shown when blocking",
  "suppressOutput": false,
  "decision": "block",
  "reason": "Explanation for decision",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Context injected back to model"
  }
}
\`\`\`

**字段：**
- \`systemMessage\` - 向用户显示消息（所有 hooks）
- \`continue\` - 设置为 \`false\` 以阻止/停止（默认：true）
- \`stopReason\` - 当 \`continue\` 为 false 时显示的消息
- \`suppressOutput\` - 从记录中隐藏 stdout（默认：false）
- \`decision\` - PostToolUse/Stop/UserPromptSubmit hooks 的 "block"（对于 PreToolUse 已弃用，请改用 hookSpecificOutput.permissionDecision）
- \`reason\` - 决策的解释
- \`hookSpecificOutput\` - 事件特定输出（必须包含 \`hookEventName\`）：
  - \`additionalContext\` - 注入到模型上下文中的文本
  - \`permissionDecision\` - "allow"、"deny" 或 "ask"（仅 PreToolUse）
  - \`permissionDecisionReason\` - 权限决策的原因（仅 PreToolUse）
  - \`updatedInput\` - 修改的工具输入（仅 PreToolUse）

### 常见模式

**写入后自动格式化：**
\`\`\`json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_response.filePath // .tool_input.file_path' | xargs prettier --write 2>/dev/null || true"
      }]
    }]
  }
}
\`\`\`

**记录所有 bash 命令：**
\`\`\`json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command' >> ~/.claude/bash-log.txt"
      }]
    }]
  }
}
\`\`\`

**向用户显示消息的停止 hook：**

命令必须输出带有 \`systemMessage\` 字段的 JSON：
\`\`\`bash
# 输出以下内容的示例命令：{"systemMessage": "Session complete!"}
echo '{"systemMessage": "Session complete!"}'
\`\`\`

**代码更改后运行测试：**
\`\`\`json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path // .tool_response.filePath' | grep -E '\\\\.(ts|js)$' && npm test || true"
      }]
    }]
  }
}
\`\`\`
