<!--
name: 'Skill: Update Claude Code Config'
description: Skill for modifying Claude Code configuration file (settings.json).
ccVersion: 2.1.76
variables:
  - SETTINGS_FILE_LOCATION_PROMPT
  - HOOKS_CONFIGURATION_PROMPT
-->
# 更新配置技能

通过更新 settings.json 文件来修改 Claude Code 配置。

## 何时需要 Hooks(非内存)

如果用户希望某些事情作为对事件的响应自动发生,他们需要在 settings.json 中配置 **hook**。内存/偏好设置无法触发自动化操作。

**这些需要 hooks:**
- "在压缩之前,问我需要保留什么" → PreCompact hook
- "编写文件后,运行 prettier" → PostToolUse hook,使用 Write|Edit 匹配器
- "当我运行 bash 命令时,记录它们" → PreToolUse hook,使用 Bash 匹配器
- "代码更改后始终运行测试" → PostToolUse hook

**Hook 事件:** PreToolUse, PostToolUse, PreCompact, PostCompact, Stop, Notification, SessionStart

## 关键:写入前先读取

**始终在更改之前读取现有的设置文件。**合并新设置与现有设置 - 永远不要替换整个文件。

## 关键:对歧义使用 AskUserQuestion

当用户的请求有歧义时,使用 AskUserQuestion 来澄清:
- 要修改哪个设置文件(用户/项目/本地)
- 是添加到现有数组还是替换它们
- 存在多个选项时的特定值

## 决策:配置工具 vs 直接编辑

**对以下简单设置使用配置工具 Config tool:**
- \`theme\`, \`editorMode\`, \`verbose\`, \`model\`
- \`language\`, \`alwaysThinkingEnabled\`
- \`permissions.defaultMode\`

**直接编辑 settings.json 用于:**
- Hooks(PreToolUse, PostToolUse 等)
- 复杂的权限规则(allow/deny 数组)
- 环境变量
- MCP 服务器配置
- 插件配置

## 工作流程

1. **澄清意图** - 如果请求有歧义则询问
2. **读取现有文件** - 在目标设置文件上使用 Read 工具
3. **小心合并** - 保留现有设置,特别是数组
4. **编辑文件** - 使用 Edit 工具(如果文件不存在,先要求用户创建)
5. **确认** - 告诉用户更改了什么

## 合并数组(重要!)

添加到权限数组或 hook 数组时,**与现有合并**,不要替换:

**错误**(替换现有权限):
\`\`\`json
{ "permissions": { "allow": ["Bash(npm:*)"] } }
\`\`\`

**正确**(保留现有 + 添加新):
\`\`\`json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",      // 现有
      "Edit(.claude)",    // 现有
      "Bash(npm:*)"       // 新
    ]
  }
}
\`\`\`

${SETTINGS_FILE_LOCATION_PROMPT}

${HOOKS_CONFIGURATION_PROMPT}

## 示例工作流程

### 添加 Hook

用户:"Claude 编写代码后格式化我的代码"

1. **澄清**: 哪种格式化程序?(prettier、gofmt 等)
2. **读取**: \`.claude/settings.json\`(如果缺失则创建)
3. **合并**: 添加到现有 hooks,不要替换
4. **结果**:
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

### 添加权限

用户:"允许 npm 命令而不提示"

1. **读取**: 现有权限
2. **合并**: 添加 \`Bash(npm:*)\` 到 allow 数组
3. **结果**: 与现有 allows 组合

### 环境变量

用户:"设置 DEBUG=true"

1. **决定**: 用户设置(全局)还是项目设置?
2. **读取**: 目标文件
3. **合并**: 添加到 env 对象
\`\`\`json
{ "env": { "DEBUG": "true" } }
\`\`\`

## 常见错误

1. **替换而不是合并** - 始终保留现有设置
2. **错误的文件** - 如果范围不清楚询问用户
3. **无效的 JSON** - 更改后验证语法
4. **忘记先读取** - 始终写入前先读取

## 故障排除 Hooks

如果 hook 没有运行:
1. **检查设置文件** - 读取 ~/.claude/settings.json 或 .claude/settings.json
2. **验证 JSON 语法** - 无效的 JSON 会静默失败
3. **检查匹配器** - 是否匹配工具名称?(例如 "Bash"、"Write"、"Edit")
4. **检查 hook 类型** - 是 "command"、"prompt" 还是 "agent"?
5. **测试命令** - 手动运行 hook 命令以查看它是否工作
6. **使用 --debug** - 运行 \`claude --debug\` 查看 hook 执行日志
