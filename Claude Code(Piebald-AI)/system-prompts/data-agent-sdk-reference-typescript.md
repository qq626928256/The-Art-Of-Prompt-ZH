<!--
name: 'Data: Agent SDK reference — TypeScript'
description: TypeScript Agent SDK reference including installation, quick start, custom tools, and hooks
ccVersion: 2.1.73
-->
# Agent SDK — TypeScript

Claude Agent SDK 提供了一个更高级别的接口，用于构建具有内置工具、安全特性和代理能力的 AI 代理。

## 安装

\`\`\`bash
npm install @anthropic-ai/claude-agent-sdk
\`\`\`

---

## 快速开始

\`\`\`typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "解释这个代码库",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
  }
}
\`\`\`

---

## 内置工具

| 工具      | 描述                          |
| --------- | ------------------------------------ |
| Read      | 读取工作区中的文件          |
| Write     | 创建新文件                     |
| Edit      | 对现有文件进行精确编辑 |
| Bash      | 执行 shell 命令               |
| Glob      | 按模式查找文件                |
| Grep      | 按内容搜索文件              |
| WebSearch | 搜索网络信息       |
| WebFetch        | 获取和分析网页          |
| AskUserQuestion | 向用户提出澄清问题         |
| Agent           | 生成子代理                      |

---

## 权限系统

\`\`\`typescript
for await (const message of query({
  prompt: "重构身份验证模块",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

权限模式：

- \`"default"\`: 为危险操作提示
- \`"plan"\`: 仅规划，不执行
- \`"acceptEdits"\`: 自动接受文件编辑
- \`"dontAsk"\`: 不提示 — **拒绝**任何未预先批准的操作（不是自动批准模式）
- \`"bypassPermissions"\`: 跳过所有提示（需要在 options 中设置 \`allowDangerouslySkipPermissions: true\`）

---

## MCP（模型上下文协议）支持

\`\`\`typescript
for await (const message of query({
  prompt: "打开 example.com 并描述你看到的",
  options: {
    mcpServers: {
      playwright: { command: "npx", args: ["@playwright/mcp@latest"] },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

### 进程内 MCP 工具

您可以使用 \`tool()\` 和 \`createSdkMcpServer\` 定义在进程中运行的自定义工具：

\`\`\`typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const myTool = tool("my-tool", "描述", { input: z.string() }, async (args) => {
  return { content: [{ type: "text", text: "结果" }] };
});

const server = createSdkMcpServer({ name: "my-server", tools: [myTool] });

// 传递给 query
for await (const message of query({
  prompt: "使用 my-tool 执行操作",
  options: { mcpServers: { myServer: server } },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

---

## 钩子

\`\`\`typescript
import { query, HookCallback } from "@anthropic-ai/claude-agent-sdk";
import { appendFileSync } from "fs";

const logFileChange: HookCallback = async (input) => {
  const filePath = (input as any).tool_input?.file_path ?? "unknown";
  appendFileSync(
    "./audit.log",
    \`\${new Date().toISOString()}: 修改了 \${filePath}\\n\`,
  );
  return {};
};

for await (const message of query({
  prompt: "重构 utils.py 以提高可读性",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits",
    hooks: {
      PostToolUse: [{ matcher: "Edit|Write", hooks: [logFileChange] }],
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

工具生命周期事件的钩子事件输入（\`PreToolUse\`、\`PostToolUse\`、\`PostToolUseFailure\`）包括 \`agent_id\` 和 \`agent_type\` 字段，允许钩子识别是哪个代理（主代理或子代理）触发了工具调用。

可用的钩子事件：\`PreToolUse\`、\`PostToolUse\`、\`PostToolUseFailure\`、\`Notification\`、\`UserPromptSubmit\`、\`SessionStart\`、\`SessionEnd\`、\`Stop\`、\`SubagentStart\`、\`SubagentStop\`、\`PreCompact\`、\`PermissionRequest\`、\`Setup\`、\`TeammateIdle\`、\`TaskCompleted\`、\`ConfigChange\`、\`Elicitation\`、\`ElicitationResult\`、\`WorktreeCreate\`、\`WorktreeRemove\`、\`InstructionsLoaded\`

---

## 常用选项

\`query()\` 接受顶层 \`prompt\`（字符串）和 \`options\` 对象：

\`\`\`typescript
query({ prompt: "...", options: { ... } })
\`\`\`

| 选项                              | 类型   | 描述                                                                |
| ----------------------------------- | ------ | -------------------------------------------------------------------------- |
| \`cwd\`                               | string | 文件操作的工作目录                                      |
| \`allowedTools\`                      | array  | 代理可以使用的工具（例如，\`["Read", "Edit", "Bash"]\`）                |
| \`tools\`                             | array \\| preset | 可用的内置工具（\`string[]\` 或 \`{type:'preset', preset:'claude_code'}\`) |
| \`disallowedTools\`                   | array  | 明确禁止的工具                                               |
| \`permissionMode\`                    | string | 如何处理权限提示                                           |
| \`allowDangerouslySkipPermissions\`   | bool   | 必须为 \`true\` 才能使用 \`permissionMode: "bypassPermissions"\`                |
| \`mcpServers\`                        | object | 要连接的 MCP 服务器                                                  |
| \`hooks\`                             | object | 用于自定义行为的钩子                                             |
| \`systemPrompt\`                      | string \\| preset | 自定义系统提示（\`string\` 或 \`{type:'preset', preset:'claude_code', append?:string}\`) |
| \`maxTurns\`                          | number | 停止前的最大代理轮次                                        |
| \`maxBudgetUsd\`                      | number | 查询的最大预算（美元）                                        |
| \`model\`                             | string | 模型 ID（默认：由 CLI 确定）                                      |
| \`agents\`                            | object | 子代理定义（\`Record<string, AgentDefinition>\`）                   |
| \`outputFormat\`                      | object | 结构化输出模式                                                   |
| \`thinking\`                          | object | 思考/推理控制                                                 |
| \`betas\`                             | array  | 要启用的 Beta 功能（例如，\`["context-1m-2025-08-07"]\`）               |
| \`settingSources\`                    | array  | 要加载的设置（例如，\`["project"]\`）。默认：无（无 CLAUDE.md 文件） |
| \`env\`                               | object | 为会话设置的环境变量                               |

---

## 子代理

\`\`\`typescript
for await (const message of query({
  prompt: "使用 code-reviewer 代理审查此代码库",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Agent"],
    agents: {
      "code-reviewer": {
        description: "专家级代码审查员，用于质量和安全审查。",
        prompt: "分析代码质量并提出改进建议。",
        tools: ["Read", "Glob", "Grep"],
      },
    },
  },
})) {
  if ("result" in message) console.log(message.result);
}
\`\`\`

---

## 消息类型

\`\`\`typescript
for await (const message of query({
  prompt: "查找 TODO 注释",
  options: { allowedTools: ["Read", "Glob", "Grep"] },
})) {
  if ("result" in message) {
    console.log(message.result);
    console.log(\`停止原因：\${message.stop_reason}\`); // 例如，"end_turn"、"tool_use"、"max_tokens"
  } else if (message.type === "system" && message.subtype === "init") {
    const sessionId = message.session_id; // 捕获以便稍后恢复
  }
}
\`\`\`

还会为子代理操作发出与任务相关的系统消息：
- \`task_started\` — 子代理任务注册时发出
- \`task_progress\` — 带有累积使用指标、工具计数和持续时间的实时进度更新
- \`task_notification\` — 任务完成通知（包括 \`tool_use_id\` 用于与原始工具调用关联）

---

## 会话历史

检索过去的会话数据：

\`\`\`typescript
import { listSessions, getSessionMessages } from "@anthropic-ai/claude-agent-sdk";

// 列出所有过去的会话
const sessions = await listSessions();
for (const session of sessions) {
  console.log(\`\${session.sessionId}: \${session.cwd}\`);
}

// 从特定会话获取消息（支持通过 limit/offset 分页）
const messages = await getSessionMessages(sessionId, { limit: 50, offset: 0 });
for (const msg of messages) {
  console.log(msg);
}
\`\`\`

---

## MCP 服务器管理

在运行的查询上运行时管理 MCP 服务器：

\`\`\`typescript
// 重新连接已断开的 MCP 服务器
await queryHandle.reconnectMcpServer("my-server");

// 打开/关闭 MCP 服务器
await queryHandle.toggleMcpServer("my-server", false);  // (name, enabled) — 两者都需要

// 获取所有配置的 MCP 服务器的状态 — 返回数组
const statuses: McpServerStatus[] = await queryHandle.mcpServerStatus();
for (const s of statuses) {
  console.log(s.name, s.scope, s.tools.length, s.error);
}
\`\`\`

---

## 最佳实践

1. **始终指定 allowedTools** — 明确列出代理可以使用的工具
2. **设置工作目录** — 始终为文件操作指定 \`cwd\`
3. **使用适当的权限模式** — 从 \`"default"\` 开始，仅在需要时升级
4. **处理所有消息类型** — 检查 \`result\` 属性以获取代理输出
5. **限制 maxTurns** — 使用合理的限制防止代理失控
