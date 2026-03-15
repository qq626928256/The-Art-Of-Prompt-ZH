<!--
name: 'Skill: /stuck slash command'
description: Diagnozse frozen or slow Claude Code sessions
ccVersion: 2.1.74
-->
# /stuck — 诊断冻结/缓慢的 Claude Code 会话

用户认为这台机器上的另一个 Claude Code 会话已冻结、卡住或非常慢。调查并向 #claude-code-feedback 发布报告。

## 寻找什么

扫描其他 Claude Code 进程(排除当前进程 — PID 在 \`process.pid\` 中,但对于 shell 命令只需排除你看到运行此提示的 PID)。进程名称通常是 \`claude\`(已安装)或 \`cli\`(本机开发构建)。

会话卡住的迹象:
- **高 CPU(≥90%)持续** — 可能是无限循环。采样两次,相隔 1-2 秒,以确认它不是瞬态尖峰。
- **进程状态 \`D\`(不间断睡眠)** — 通常是 I/O 挂起。\`ps\` 输出中的 \`state\` 列;第一个字符很重要(忽略修饰符如 \`+\`、\`s\`、\`<\`)。
- **进程状态 \`T\`(已停止)** — 用户可能意外按了 Ctrl+Z。
- **进程状态 \`Z\`(僵尸)** — 父进程未回收。
- **非常高的 RSS(≥4GB)** — 可能的内存泄漏使会话缓慢。
- **卡住的子进程** — 挂起的 \`git\`、\`node\` 或 shell 子进程可以使父进程冻结。检查每个会话的 \`pgrep -lP <pid>\`。

## 调查步骤

1. **列出所有 Claude Code 进程**(macOS/Linux):
   ```
   ps -axo pid=,pcpu=,rss=,etime=,state=,comm=,command= | grep -E '(claude|cli)' | grep -v grep
   ```
   筛选 \`comm\` 为 \`claude\` 或(\`cli\` 且命令路径包含 "claude")的行。

2. **对于任何可疑的内容**,收集更多上下文:
   - 子进程:\`pgrep -lP <pid>\`
   - 如果高 CPU:1-2 秒后再次采样以确认它是持续的
   - 如果子进程看起来挂起(例如 git 命令),使用 \`ps -p <child_pid> -o command=\` 记录其完整命令行
   - 如果可以推断会话 ID,检查会话的调试日志:\`~/.claude/debug/<session-id>.txt\`(最后几百行通常显示挂起前它在做什么)

3. **考虑堆栈转储**对于真正冻结的进程(高级,可选):
   - macOS:\`sample <pid> 3\` 给出 3 秒的本机堆栈采样
   - 这很大 — 仅在进程明显挂起且你想知道*原因*时才抓取

## 报告

使用 Slack MCP 工具向 **#claude-code-feedback**(频道 ID:\`C07VBSHV7EV\`)发布摘要。如果 \`slack_send_message\` 尚未加载,使用 ToolSearch 查找它。

报告应包括:
- 主机名、Claude Code 版本、总会话数、看起来卡住的会话数
- 对于每个标记的会话:PID、CPU%、RSS、状态、运行时间、命令行、子进程以及你对可能出错的诊断
- 如果没有标记任何内容,仍然发布简短的"一切正常"以及会话计数 — 用户运行 /stuck 是有原因的,因此确认"从外部看起来一切正常"是有用的

如果 Slack MCP 不可用,将报告格式化为用户可以复制粘贴到 #claude-code-feedback 的消息。

## 注意
- 不要杀死或向任何进程发送信号 — 这仅用于诊断。
- 在 Slack 消息中要简洁;详细信息可以放在代码块中。
- 如果用户给出了参数(例如特定的 PID 或症状),首先关注那里。
