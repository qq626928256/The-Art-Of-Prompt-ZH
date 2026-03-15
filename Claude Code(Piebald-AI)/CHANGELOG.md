<!--
Note: Only use **NEW:** for entirely new prompt files, NOT for new additions/sections within existing prompts.
-->

### Claude Code 系统提示词更新日志

# [2.1.76](https://github.com/Piebald-AI/claude-code-system-prompts/commit/6cc7a81)

_+43 tokens_

- Agent Prompt: Security monitor for autonomous agent actions (second part) — 将敏感数据检测中的 "base64-encoded" 澄清为 "encoded (e.g. base64)"；将来自外部的反序列化代码示例扩展为 "可以执行代码的格式 (eval, exec, yaml.unsafe_load, pickle, 等)"；通过移除 "model registrations" 完善了 "Modify Shared Resources" 示例；改进了 "Irreversible Local Destruction" 格式并澄清了包管理器控制的目录指导（说明文件会在安装时重新生成，并建议复制到源代码树中）；在 External System Writes 中将 "GitHub issues/PRs" 的大小写改为 "GitHub Issues/PRs"；更新了 Data Exfiltration，将 "creating gists" 替换为 "公共明文共享应用程序 (例如公共 GitHub gists)"；在交叉引用中为规则名称添加了引号（例如 "Local Operations" ALLOW 异常，BLOCK 中的 "Irreversible Local Destruction"）。
- Skill: Update Claude Code Config — 在可用 hook 事件列表中添加了 `PostCompact`。
- System Prompt: Hooks Configuration — 在 hooks 事件表中添加了 `PostCompact` hook 事件（压缩后触发，接收摘要）。
- Tool Description: ReadFile — 精简并重新排序了使用说明；添加了关于读取完整文件的说明。


# [2.1.75](https://github.com/Piebald-AI/claude-code-system-prompts/commit/97ce0c2)

_+156 tokens_

- **NEW:** Agent Prompt: Determine which memory files to attach — 用于确定主代理要附加哪些内存文件的 Agent。
- Skill: Update Claude Code Config — 在可用 hook 事件列表中添加了 `OnRequestComplete` 和 `OnRequestFailed`。
- System Prompt: Hooks Configuration — 在 hooks 事件表中添加了 `OnRequestComplete`（在请求完成时触发，接收响应）和 `OnRequestFailed`（在请求失败时触发，接收错误）事件。


# [2.1.74](https://github.com/Piebald-AI/claude-code-system-prompts/commit/6725e86)

_+10 tokens_

- Tool Description: Bash — 更新了说明，改为 "避免使用此工具运行 `find`、`grep`、`cat`、`head`、`tail`、`sed`、`awk` 或 `echo` 命令"，并列出了相应的专用工具。


# [2.1.73](https://github.com/Piebald-AI/claude-code-system-prompts/commit/cfa4f1a)

_+77 tokens_

- Agent Prompt: Security monitor for autonomous agent actions (second part) — 澄清了文件删除安全性检查，删除不应被视为本地操作（例如 `rm -rf node_modules`）；添加了关于通过检查 `https://platform.openai.com` 域来检测 OpenAI SDK 用法的说明；移除了 `/v1/` 路径检查，因为它过于严格；添加了检查字符串 "news" 以更可靠地捕获 News tool；将 "new credit card" 示例更改为 "credit card info" 以避免与 "new" 关键字冲突。
- Tool Description: Bash — 扩展了 "检查输出" 示例，改为使用 `&&` 串联的命令；更改为推荐将她的命令添加到单个消息中以提高性能。


# [2.1.72](https://github.com/Piebald-AI/claude-code-system-prompts/commit/2f79b18)

_+96 tokens_

- Agent Prompt: Security monitor for autonomous agent actions (second part) — 添加了关于全局创建新文件（例如在任何地方创建 `~/.ssh/authorized_keys`）的安全检查；更频繁地检查每个工具调用；阐明了 "write to the home directory" 与 "write to the current working directory"（取决于 cwd，写入家目录可能是写入项目目录）；更新了删除检查以捕获确切的目录路径（例如 `/Users/user/.ssh`），而不仅仅是 `/Users/user`；更清楚地解释了为什么 `sed -i` 是危险操作（它破坏符号链接并可能导致权限错误）；澄清了 "Use multiple tools in a single response" 是指 "Use multiple tool calls"。
- Tool Description: Bash — 扩展了说明，允许在其命令中使用 `&` 后台语法；添加了示例 "长期运行并希望稍后获得完成通知 — 使用此参数"。
- Tool Description: Edit — 添加了关于 "preserving exact indentation" 的说明，以强调需要匹配空格/制表符。


# [2.1.71](https://github.com/Piebald-AI/claude-code-system-prompts/commit/ce16b8f)

_+207 tokens_

- **NEW:** Agent Prompt: Security monitor for autonomous agent actions (second part) — 第二部分（第一个 "BLOCK" 部分），通过在代理执行每个操作之前检查来监控代理行为。检查可疑/危险操作，包括：在目标目录或家目录之外的本地操作、在 cwd 之外创建文件、删除文件/目录、写入外部系统、使用高风险关键字（新闻、电子邮件、密码、API 密钥、令牌、加密等）、调用其他 LLM、连接到远程主机、运行后台/无限循环、数据外泄、从外部反序列化代码、上传文件、下载文件、git reset --hard、git push --force、fork 炸弹、fork/clone bomb、删除二进制文件、在共享资源上使用 git 命令、删除程序、在 cwd 之外编辑文件、修改共享资源、启动交互式 shell、SSH-ing、复制/粘贴、使用编码数据（base64 等）。
- Agent Prompt: Security monitor for autonomous agent actions — 重命名为 "Security monitor for autonomous agent actions (first part)" 并说明该部分适用于*请求的*操作。
- Agent Prompt: Task planning — 在计划中添加了对 "Security monitoring" 的引用。


# [2.1.70](https://github.com/Piebald-AI/claude-code-system-prompts/commit/74518a3)

_+63 tokens_

- Agent Prompt: Task planning — 添加了说明 "ALWAYS 使用 TodoWrite 工具在以下情况下创建和结构化任务列表：用户明确要求使用待办事项列表，用户提供了多个任务（编号或逗号分隔），收到新指令时，开始执行任务时，完成任务时"。移除了重复的 "不创建待办事项列表" 指导。
- Tool Description: TodoWrite — 添加了使用场景 "用户明确要求使用待办事项列表"。


# [2.1.69](https://github.com/Piebald-AI/claude-code-system-prompts/commit/4b012b9)

_+23 tokens_

- Tool Description: Bash — 添加了说明 "可以使用 `run_in_background` 参数在后台运行命令"。


# [2.1.68](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d8d2e30)

_+166 tokens_

- Tool Description: Bash — 扩展了关于复杂命令需要更多上下文的说明，示例包括管道命令、模糊标志和 "难以一眼解析的内容"。添加了避免在 git 命令中使用 `-i` 标志的说明，因为它需要交互式输入。
- Tool Description: Edit — 添加了说明，如果 `old_string` 在文件中不唯一，编辑将失败，并建议使用更大的上下文或 `replace_all` 参数。
- Tool Description: Write — 澄清了写入现有文件时的差异。


# [2.1.67](https://github.com/Piebald-AI/claude-code-system-prompts/commit/2a93f18)

_+326 tokens_

- Agent Prompt: Committing changes with git — 大幅重新组织并扩展了 "Committing changes with git" 部分。添加了关于预提交 hook 的详细说明、如何处理 hook 失败、为什么 `--amend` 是危险的，以及如何使用 heredoc 编写多行提交消息的示例。更新了 Git 提交示例以使用 heredoc 语法。
- Tool Description: Bash — 添加了说明 "不要使用换行符分隔命令（在引用字符串中可以）。"
- Tool Description: Edit — 更新了说明以匹配 Bash 工具的措辞。


# [2.1.66](https://github.com/Piebald-AI/claude-code-system-prompts/commit/54a820e)

_+53 tokens_

- Agent Prompt: Committing changes with git — 添加了 "重要说明：" 部分，包括：不要在 hook 失败时使用 `--amend`，hook 失败意味着提交未发生，因此 `--amend` 会修改*上一个*提交；不要使用 `-A` 标志来避免意外提交敏感文件；在提交前创建提交（不要先读取/探索代码）。
- Tool Description: Bash — 添加了关于使用 `&&` vs `;` 链接命令的说明。


# [2.1.65](https://github.com/Piebald-AI/claude-code-system-prompts/commit/1a1f9ff)

_+14 tokens_

- Tool Description: Bash — 添加了说明 "如果命令将创建新目录或文件，请先使用此工具运行 `ls` 以验证父目录存在且位置正确。"


# [2.1.64](https://github.com/Piebald-AI/claude-code-system-prompts/commit/8113c74)

_+7 tokens_

- Tool Description: Bash — 添加了示例 "git reset --hard origin/main → 丢弃所有本地更改并与远程 main 匹配"。


# [2.1.63](https://github.com/Piebald-AI/claude-code-system-prompts/commit/53f83d3)

_+13 tokens_

- Tool Description: Bash — 添加了示例 "git checkout -b → 创建新分支"。


# [2.1.62](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d4a0d89)

_+34 tokens_

- Tool Description: Bash — 添加了说明 "在创建拉取请求时，请使用 gh 命令。"
- Tool Description: Bash — 添加了示例 "git status → 显示工作树状态"。


# [2.1.61](https://github.com/Piebald-AI/claude-code-system-prompts/commit/4c32e6f)

_+134 tokens_

- Agent Prompt: Committing changes with git — 更新了创建提交的步骤。步骤 1 更改为并行运行三个 git 命令。步骤 2 添加了 "不要提交可能包含机密的文件（.env、credentials.json 等）。如果用户特别要求提交这些文件，请警告用户"。步骤 3 更改为并行运行三个命令（git add、git commit、git status）。
- Tool Description: Bash — 更新了说明，建议 "可以调用多个工具"。添加了示例 "运行 git status 命令..."。添加了说明 "对于 git 命令："，后跟 3 个 git 特定的要点。
- Tool Description: Bash — 添加了说明 "优先添加特定文件..." 后跟示例。


# [2.1.60](https://github.com/Piebald-AI/claude-code-system-prompts/commit/963dc92)

_+2 tokens_

- Agent Prompt: Task planning — 更改了示例 "用户要求我实现这些功能..." 以说明代理推断了待办事项列表。


# [2.1.59](https://github.com/Piebald-AI/claude-code-system-prompts/commit/31a9d7f)

_+5 tokens_

- Agent Prompt: Task planning — 更改了示例 "用户要求我优化..." 以从分析中推断出待办事项列表。


# [2.1.58](https://github.com/Piebald-AI/claude-code-system-prompts/commit/3d45fe0)

_+87 tokens_

- Agent Prompt: Task planning — 重新组织了 "何时使用此工具" 部分。扩展了 "用户提供了多个任务" 示例，以说明每个功能的任务分解。
- Agent Prompt: Task planning — 更改了 "单个直接任务" 示例以显示 "打印 Hello World"。


# [2.1.57](https://github.com/Piebald-AI/claude-code-system-prompts/commit/2aa2d61)

_+36 tokens_

- Agent Prompt: Task planning — 添加了新的示例场景 "用户提供了多个任务"。更新了 "用户明确要求使用待办事项列表" 示例。
- Agent Prompt: Task planning — 更改了 "用户提供多个任务" 示例以使用功能列表而不是编号列表。


# [2.1.56](https://github.com/Piebald-AI/claude-code-system-prompts/commit/1e74a61)

_+2 tokens_

- Tool Description: Bash — 更改了 `npm install` 示例描述为 "安装包依赖项"。


# [2.1.55](https://github.com/Piebald-AI/claude-code-system-prompts/commit/a1030de)

_+94 tokens_

- Agent Prompt: Task planning — 扩展了 "何时 NOT 使用此工具" 部分，添加了更多示例场景。添加了示例说明何时使用待办事项列表（例如添加暗黑模式切换）。
- Tool Description: Bash — 扩展了说明以包括 "复杂命令" 和 "难以一眼解析的内容"。


# [2.1.54](https://github.com/Piebald-AI/claude-code-system-prompts/commit/a67a1fa)

_+4 tokens_

- Agent Prompt: Task planning — 更新了 "用户明确要求使用待办事项列表" 示例，以移除 "运行测试"。


# [2.1.53](https://github.com/Piebald-AI/claude-code-system-prompts/commit/a45ea72)

_+31 tokens_

- Agent Prompt: Task planning — 扩展了 "何时使用此工具" 部分，添加了更多示例场景。更新了 "用户明确要求使用待办事项列表" 示例。


# [2.1.52](https://github.com/Piebald-AI/claude-code-system-prompts/commit/9d0e981)

_+33 tokens_

- Agent Prompt: Task planning — 添加了说明 "在完成任务后 — 立即标记为完成（不要批量完成）。"


# [2.1.51](https://github.com/Piebald-AI/claude-code-system-prompts/commit/4dc4c11)

_+55 tokens_

- Agent Prompt: Task planning — 更新了说明以同时移除 "角色" 和 "格式"。添加了说明 "始终提供一个任务处于进行中（不能更少，也不能更多）。"


# [2.1.50](https://github.com/Piebald-AI/claude-code-system-prompts/commit/6e6b2b6)

_+3 tokens_

- Agent Prompt: Task planning — 添加了 "activeForm" 到任务状态和管理的说明中。


# [2.1.49](https://github.com/Piebald-AI/claude-code-system-prompts/commit/aa626d1)

_+103 tokens_

- Agent Prompt: Task planning — 添加了关于任务状态和管理的新部分。添加了说明 "移除不再相关的任务"。
- Agent Prompt: Task planning — 添加了说明 "任务描述必须有两种形式：content（命令式形式）和 activeForm（现在进行时形式）。"
- Tool Description: TodoWrite — 更新了 `activeForm` 参数说明。


# [2.1.48](https://github.com/Piebald-AI/claude-code-system-prompts/commit/fc33544)

_+28 tokens_

- Agent Prompt: Task planning — 添加了说明 "添加在实现过程中发现的任何新后续任务"。


# [2.1.47](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d70bc64)

_+3 tokens_

- Agent Prompt: Task planning — 添加了 "系统提示词" 部分。


# [2.1.46](https://github.com/Piebald-AI/claude-code-system-prompts/commit/90c0867)

_+5 tokens_

- Agent Prompt: Task planning — 澄清了计划步骤的说明 "创建具体、可操作的项目"。


# [2.1.45](https://github.com/Piebald-AI/claude-code-system-prompts/commit/42f82c8)

_+8 tokens_

- Agent Prompt: Task planning — 澄清了计划步骤的说明 "为每个功能创建具体任务"。


# [2.1.44](https://github.com/Piebald-AI/claude-code-system-prompts/commit/526894e)

_+86 tokens_

- Agent Prompt: Task planning — 添加了新部分 "任务分解"。添加了说明 "将复杂的任务分解为更小的、可管理的步骤"。添加了示例 "用户要求我实现这些功能..."。
- Agent Prompt: Task planning — 更新了 "何时使用此工具" 示例 #3 以说明多步骤功能请求。


# [2.1.43](https://github.com/Piebald-AI/claude-code-system-prompts/commit/696276a)

_+7 tokens_

- Agent Prompt: Task planning — 更新了 "何时使用此工具" 示例 #3 以使用多步骤任务描述。


# [2.1.42](https://github.com/Piebald-AI/claude-code-system-prompts/commit/66e66a5)

_+29 tokens_

- Agent Prompt: Task planning — 添加了说明 "在开始工作之前标记为 in_progress"。添加了说明 "理想情况下，一次应该只有一个待办事项处于 in_progress 状态"。


# [2.1.41](https://github.com/Piebald-AI/claude-code-system-prompts/commit/f6c2dc5)

_+93 tokens_

- Tool Description: Bash — 添加了说明 "对每个响应始终使用绝对路径"。
- Tool Description: Edit — 添加了说明 "此工具需要绝对路径，不能是相对路径"。
- Tool Description: Write — 添加了说明 "file_path 参数必须是绝对路径，不能是相对路径"。


# [2.1.40](https://github.com/Piebald-AI/claude-code-system-prompts/commit/1f19a2b)

_+26 tokens_

- Agent Prompt: Task planning — 添加了示例 "用户要求我优化他们的 React 应用程序..."。更新了何时 NOT 使用待办事项列表的示例 #1。


# [2.1.39](https://github.com/Piebald-AI/claude-code-system-prompts/commit/1af71c0)

_+11 tokens_

- Agent Prompt: Task planning — 添加了示例 "用户询问如何打印 Hello World..."。


# [2.1.38](https://github.com/Piebald-AI/claude-code-system-prompts/commit/32cf5c0)

_+48 tokens_

- Agent Prompt: Task planning — 扩展了 "何时 NOT 使用此工具" 部分，添加了更多示例场景。
- Agent Prompt: Task planning — 更新了 "何时使用此工具" 示例 #3 以显示多步骤任务。


# [2.1.37](https://github.com/Piebald-AI/claude-code-system-prompts/commit/16ee54c)

_+6 tokens_

- Agent Prompt: Task planning — 更新了 "何时使用此工具" 示例 #2 以使用单个简单的命令执行任务。


# [2.1.36](https://github.com/Piebald-AI/claude-code-system-prompts/commit/5a56108)

_+11 tokens_

- Agent Prompt: Task planning — 更新了示例 #2 以展示简单的信息请求。


# [2.1.35](https://github.com/Piebald-AI/claude-code-system-prompts/commit/c68196d)

_+60 tokens_

- Agent Prompt: Task planning — 添加了新示例 #2（信息性请求）。
- Agent Prompt: Task planning — 更新了示例 #1 以显示简单任务。


# [2.1.34](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d20b915)

_+43 tokens_

- Agent Prompt: Task planning — 添加了 "示例" 部分，包含 1 个示例。
- Agent Prompt: Task planning — 扩展了 "何时 NOT 使用此工具" 部分，添加了第 2 个条件。


# [2.1.33](https://github.com/Piebald-AI/claude-code-system-prompts/commit/f9ce22d)

_+109 tokens_

- Agent Prompt: Task planning — 添加了 "何时使用此工具" 部分，包含 3 个条件。
- Agent Prompt: Task planning — 添加了 "何时 NOT 使用此工具" 部分，包含 1 个条件。


# [2.1.32](https://github.com/Piebald-AI/claude-code-system-prompts/commit/e984dc9)

_+49 tokens_

- Agent Prompt: Task planning — 更新了说明，使用 "ActiveForm" 而不是 "进行时形式"。添加了新示例 "用户提供了多个任务"。添加了说明 "不要批量完成"。


# [2.1.31](https://github.com/Piebald-AI/claude-code-system-prompts/commit/358df93)

_+115 tokens_

- Agent Prompt: Task planning — 添加了 "用户提供了多个任务" 到使用场景列表中。
- Agent Prompt: Task planning — 添加了关于收到新指令后立即捕获用户要求的说明。


# [2.1.30](https://github.com/Piebald-AI/claude-code-system-prompts/commit/9409383)

_+35 tokens_

- Agent Prompt: Task planning — 添加了 "用户明确要求使用待办事项列表" 到使用场景列表中。


# [2.1.29](https://github.com/Piebald-AI/claude-code-system-prompts/commit/ed15785)

_+31 tokens_

- Agent Prompt: Task planning — 澄清了简单直接任务与复杂多步骤任务的区别。


# [2.1.28](https://github.com/Piebald-AI/claude-code-system-prompts/commit/06a495c)

_+115 tokens_

- Agent Prompt: Task planning — 重写了 "何时使用 TodoWrite 工具" 部分，包括 4 个场景（复杂多步骤任务、非平凡和复杂的任务、用户明确要求、用户提供多个任务）。
- Tool Description: TodoWrite — 将长描述拆分为概述和使用场景列表。


# [2.1.27](https://github.com/Piebald-AI/claude-code-system-prompts/commit/38533bc)

_+28 tokens_

- Agent Prompt: Task planning — 添加了说明，提醒在开始工作之前将任务标记为 in_progress，完成后立即标记为 completed。


# [2.1.26](https://github.com/Piebald-AI/claude-code-system-prompts/commit/e931a3e)

_+21 tokens_

- Agent Prompt: Task planning — 添加了说明，将 "跟踪进度" 更改为 "跟踪进度并展示对任务的彻底执行"。


# [2.1.25](https://github.com/Piebald-AI/claude-code-system-prompts/commit/29ce22b)

_+30 tokens_

- Agent Prompt: Task planning — 添加了说明，使用户能够理解任务的进度和整体进度。


# [2.1.24](https://github.com/Piebald-AI/claude-code-system-prompts/commit/e8936fa)

_+42 tokens_

- Tool Description: TodoWrite — 添加了 `activeForm` 参数。


# [2.1.23](https://github.com/Piebald-AI/claude-code-system-prompts/commit/53f4432)

_+30 tokens_

- Tool Description: Bash — 更新了说明，优先使用 Glob 而不是 `find` 或 `ls`。


# [2.1.22](https://github.com/Piebald-AI/claude-code-system-prompts/commit/9ce0ed7)

_+46 tokens_

- Tool Description: Bash — 更新了说明以提及 Glob 和 Grep 工具。添加了示例 "文件搜索：使用 Glob（而不是 find 或 ls）"。


# [2.1.21](https://github.com/Piebald-AI/claude-code-system-prompts/commit/3df8b30)

_+60 tokens_

- Tool Description: Bash — 更新了说明，优先使用专用工具而不是通用 Bash 命令。


# [2.1.20](https://github.com/Piebald-AI/claude-code-system-prompts/commit/90f3c96)

_+61 tokens_

- **NEW:** Tool Description: TodoWrite — 创建和结构化任务列表的工具。
- Agent Prompt: Task planning — 添加了关于使用 TodoWrite 工具的详细说明。


# [2.1.19](https://github.com/Piebald-AI/claude-code-system-prompts/commit/a53bc7f)

_+45 tokens_

- System Prompt: Hooks Configuration — 在 hooks 事件表中添加了 `OnPreSend` 事件。


# [2.1.18](https://github.com/Piebald-AI/claude-code-system-prompts/commit/edce4ec)

_+4 tokens_

- Agent Prompt: Task planning — 澄清了应主动使用 TodoWrite 工具。


# [2.1.17](https://github.com/Piebald-AI/claude-code-system-prompts/commit/70880b9)

_+4 tokens_

- Agent Prompt: Task planning — 添加了示例说明何时使用 TodoWrite 工具。


# [2.1.16](https://github.com/Piebald-AI/claude-code-system-prompts/commit/0ef0a3f)

_+11 tokens_

- Agent Prompt: Task planning — 添加了优先考虑用户请求的说明。


# [2.1.15](https://github.com/Piebald-AI/claude-code-system-prompts/commit/38b74c9)

_+21 tokens_

- Agent Prompt: Task planning — 添加了关于使用待办事项列表的说明。


# [2.1.14](https://github.com/Piebald-AI/claude-code-system-prompts/commit/6b5ff38)

_+41 tokens_

- Agent Prompt: Codebase understanding — 添加了关于确保遵循项目说明的说明。


# [2.1.13](https://github.com/Piebald-AI/claude-code-system-prompts/commit/748d7d4)

_+23 tokens_

- Agent Prompt: Codebase understanding — 添加了关于读取项目文件的说明。


# [2.1.12](https://github.com/Piebald-AI/claude-code-system-prompts/commit/f96d9fc)

_+30 tokens_

- Agent Prompt: Codebase understanding — 添加了关于检查多个位置和考虑不同命名约定的说明。


# [2.1.11](https://github.com/Piebald-AI/claude-code-system-prompts/commit/0184198)

_+40 tokens_

- Agent Prompt: Codebase understanding — 添加了关于彻底搜索多个文件的说明。


# [2.1.10](https://github.com/Piebald-AI/claude-code-system-prompts/commit/62e0957)

_+55 tokens_

- Agent Prompt: Codebase understanding — 添加了关于从广泛的搜索开始并缩小范围的说明。


# [2.1.9](https://github.com/Piebald-AI/claude-code-system-prompts/commit/9713d85)

_+41 tokens_

- Agent Prompt: Codebase understanding — 添加了关于读取特定文件路径的说明。


# [2.1.8](https://github.com/Piebald-AI/claude-code-system-prompts/commit/5c7a46e)

_+24 tokens_

- Agent Prompt: Codebase understanding — 添加了关于使用搜索工具的说明。


# [2.1.7](https://github.com/Piebald-AI/claude-code-system-prompts/commit/4755e57)

_+31 tokens_

- Agent Prompt: Codebase understanding — 添加了关于多步骤研究任务的说明。


# [2.1.6](https://github.com/Piebald-AI/claude-code-system-prompts/commit/a89c6de)

_+22 tokens_

- Agent Prompt: Task planning — 更新了说明以使用 "多个" 而不是 "多个不同的"。


# [2.1.5](https://github.com/Piebald-AI/claude-code-system-prompts/commit/19b7772)

_+96 tokens_

- Agent Prompt: Task planning — 添加了关于复杂多步骤任务的说明。


# [2.1.4](https://github.com/Piebald-AI/claude-code-system-prompts/commit/2399178)

_+21 tokens_

- Agent Prompt: Task planning — 添加了关于跟踪进度的说明。


# [2.1.3](https://github.com/Piebald-AI/claude-code-system-prompts/commit/a05673e)

_+3 tokens_

- Tool Description: Bash — 更新了示例 `git checkout .` 描述为 "丢弃所有本地更改"。


# [2.1.2](https://github.com/Piebald-AI/claude-code-system-prompts/commit/88e3e87)

_+57 tokens_

- Tool Description: Bash — 添加了关于 `git reset --hard` 的说明。


# [2.1.1](https://github.com/Piebald-AI/claude-code-system-prompts/commit/829eb66)

_+47 tokens_

- Tool Description: Bash — 添加了关于 `git push --force` 的说明。


# [2.1.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/7fc6625)

_+336 tokens_

- **NEW:** System Prompt: Hooks Configuration — 添加了 hooks 配置系统提示词，包括事件表（`OnUserMessage`、`OnAgentMessage`、`OnToolOutput`、`OnFirstToken`、`OnRequestComplete`、`OnRequestFailed`、`OnExit`、`OnPreSend`）和配置格式。
- **NEW:** Skill: Update Claude Code Config — 用于更新 Claude Code 配置的 Skill。


# [2.0.9](https://github.com/Piebald-AI/claude-code-system-prompts/commit/a54bc80)

_+40 tokens_

- Agent Prompt: Committing changes with git — 添加了关于检查未跟踪文件的说明。


# [2.0.8](https://github.com/Piebald-AI/claude-code-system-prompts/commit/5b775c1)

_+33 tokens_

- Agent Prompt: Committing changes with git — 更新了步骤 1，以使用 `-uall` 标志运行 git status。


# [2.0.7](https://github.com/Piebald-AI/claude-code-system-prompts/commit/e226776)

_+27 tokens_

- Agent Prompt: Committing changes with git — 添加了关于在提交前检查暂存更改的说明。


# [2.0.6](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d32009e)

_+39 tokens_

- Agent Prompt: Committing changes with git — 添加了关于检查未暂存更改的说明。


# [2.0.5](https://github.com/Piebald-AI/claude-code-system-prompts/commit/47e42a8)

_+15 tokens_

- Agent Prompt: Committing changes with git — 澄清了步骤 1，使用 `git status` 和 `git diff`。


# [2.0.4](https://github.com/Piebald-AI/claude-code-system-prompts/commit/e022f5b)

_+7 tokens_

- Agent Prompt: Committing changes with git — 澄清了步骤 1 包括检查暂存的更改。


# [2.0.3](https://github.com/Piebald-AI/claude-code-system-prompts/commit/7f1f438)

_+35 tokens_

- Agent Prompt: Committing changes with git — 添加了关于查看暂存和未暂存更改的说明。


# [2.0.2](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d89cece)

_+47 tokens_

- Agent Prompt: Committing changes with git — 添加了关于使用并行 git 命令的说明。


# [2.0.1](https://github.com/Piebald-AI/claude-code-system-prompts/commit/98875a2)

_+113 tokens_

- Agent Prompt: Committing changes with git — 添加了关于并行运行命令的说明。


# [2.0.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/1a52cf6)

_+412 tokens_

- **NEW:** Agent Prompt: Committing changes with git — 添加了关于提交更改的说明，包括提交前分析更改、添加文件、创建提交和验证。
- **NEW:** Agent Prompt: Creating pull requests — 添加了关于创建 PR 的说明，包括检查分支状态、分析更改、创建 PR 和返回 URL。


# [1.16.9](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d0b94a7)

_+2 tokens_

- Tool Description: Edit — 将 "str" 更改为 "string"。


# [1.16.8](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d8c739a)

_+50 tokens_

- Tool Description: Edit — 添加了关于需要唯一 `old_string` 的说明。


# [1.16.7](https://github.com/Piebald-AI/claude-code-system-prompts/commit/3221ce3)

_+18 tokens_

- Tool Description: Read — 澄清了 PDF 文件需要 pages 参数。


# [1.16.6](https://github.com/Piebald-AI/claude-code-system-prompts/commit/bd8c466)

_+32 tokens_

- Tool Description: Read — 添加了关于读取 PDF 文件的说明。


# [1.16.5](https://github.com/Piebald-AI/claude-code-system-prompts/commit/c64fa2e)

_+29 tokens_

- Agent Prompt: Committing changes with git — 添加了关于不使用 amend 的说明。


# [1.16.4](https://github.com/Piebald-AI/claude-code-system-prompts/commit/0ecb0f3)

_+34 tokens_

- Agent Prompt: Committing changes with git — 添加了关于跳过 hooks 的说明。


# [1.16.3](https://github.com/Piebald-AI/claude-code-system-prompts/commit/5188ca9)

_+31 tokens_

- Agent Prompt: Committing changes with git — 添加了关于不使用 destructive 命令的说明。


# [1.16.2](https://github.com/Piebald-AI/claude-code-system-prompts/commit/e6fdd5f)

_+44 tokens_

- Agent Prompt: Committing changes with git — 添加了关于不更新 git 配置的说明。


# [1.16.1](https://github.com/Piebald-AI/claude-code-system-prompts/commit/5d5ebc8)

_+26 tokens_

- Agent Prompt: Committing changes with git — 添加了关于不更新 git 配置的说明。


# [1.16.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/73ba839)

_+401 tokens_

- **NEW:** Agent Prompt: Committing changes with git — 添加了关于提交更改的详细说明。
- **NEW:** Agent Prompt: Creating pull requests — 添加了关于创建 PR 的详细说明。


# [1.15.4](https://github.com/Piebald-AI/claude-code-system-prompts/commit/60f615b)

_+4 tokens_

- Agent Prompt: Creating pull requests — 添加了关于 PR 标题长度的说明。


# [1.15.3](https://github.com/Piebald-AI/claude-code-system-prompts/commit/cdc0a92)

_+32 tokens_

- Agent Prompt: Creating pull requests — 添加了关于创建分支的说明。


# [1.15.2](https://github.com/Piebald-AI/claude-code-system-prompts/commit/20eab4a)

_+54 tokens_

- Agent Prompt: Creating pull requests — 添加了关于推送分支的说明。


# [1.15.1](https://github.com/Piebald-AI/claude-code-system-prompts/commit/3f6df73)

_+34 tokens_

- Agent Prompt: Creating pull requests — 添加了关于 gh 命令的说明。


# [1.15.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/7e19f62)

_+51 tokens_

- **NEW:** Agent Prompt: Creating pull requests — 添加了关于创建 PR 的说明。


# [1.14.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/513e86b)

_+16 tokens_

- Tool Description: Bash — 添加了关于运行 `git log` 的说明。


# [1.13.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/67a1f0c)

_+16 tokens_

- Tool Description: Bash — 添加了关于运行 `git diff` 的说明。


# [1.12.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/96eb25e)

_+17 tokens_

- Tool Description: Bash — 添加了关于运行 `git status` 的说明。


# [1.11.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d93e2b2)

_+18 tokens_

- Tool Description: Bash — 添加了关于 Git 的说明。


# [1.10.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/813f68d)

_+19 tokens_

- Tool Description: Bash — 添加了关于避免使用 `sleep` 的说明。


# [1.9.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/286d4bb)

_+15 tokens_

- Tool Description: Bash — 添加了关于 `&&` 与 `;` 的说明。


# [1.8.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/0b21f8b)

_+21 tokens_

- Tool Description: Bash — 添加了关于后台进程的说明。


# [1.7.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/bb76738)

_+24 tokens_

- Tool Description: Bash — 添加了关于引用包含空格的文件路径的说明。


# [1.6.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/b36831e)

_+35 tokens_

- Tool Description: Bash — 添加了关于避免使用 `cd` 的说明。


# [1.5.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/e46c2b7)

_+18 tokens_

- Tool Description: Bash — 添加了关于使用绝对路径的说明。


# [1.4.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/65c0c1d)

_+23 tokens_

- Tool Description: Bash — 添加了关于工作目录持久化的说明。


# [1.3.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/75b3a7a)

_+38 tokens_

- Tool Description: Bash — 添加了关于 shell 状态的说明。


# [1.2.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/b45175e)

_+58 tokens_

- Tool Description: Bash — 添加了关于命令描述的说明。


# [1.1.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/08a1d29)

_+8 tokens_

- Agent Prompt: File search and codebase understanding — 添加了关于 `Glob` 的说明。


# [1.0.0](https://github.com/Piebald-AI/claude-code-system-prompts/commit/d7e1c0b)

_+37 tokens_

- **NEW:** Tool Description: Bash — 添加了 Bash 工具描述。
- **NEW:** Tool Description: Edit — 添加了 Edit 工具描述。
- **NEW:** Tool Description: Glob — 添加了 Glob 工具描述。
- **NEW:** Tool Description: Grep — 添加了 Grep 工具描述。
- **NEW:** Tool Description: Read — 添加了 Read 工具描述。
- **NEW:** Tool Description: Write — 添加了 Write 工具描述。
- **NEW:** Agent Prompt: File search and codebase understanding — 添加了文件搜索和代码库理解的说明。
- **NEW:** Agent Prompt: Task planning — 添加了任务规划的说明。
- **NEW:** Agent Prompt: Security monitor for autonomous agent actions — 添加了安全监控的说明。
- **NEW:** System Prompt: Environment information — 添加了环境信息系统提示词。
- **NEW:** System Prompt: Agent capabilities — 添加了代理能力系统提示词。
- **NEW:** System Prompt: Version information — 添加了版本信息系统提示词。
- **NEW:** System Prompt: currentDate — 添加了当前日期系统提示词。
