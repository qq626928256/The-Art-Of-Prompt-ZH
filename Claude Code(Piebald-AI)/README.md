<div>
<div align="right">
<a href="https://piebald.ai"><img width="200" top="20" align="right" src="https://github.com/Piebald-AI/.github/raw/main/Wordmark.svg"></a>
</div>

<div align="left">

### 体验 Piebald
我们发布了 **Piebald**，终极的 AI 代理开发体验。 \
免费下载并试用！ **https://piebald.ai/**

<a href="https://piebald.ai/discord"><img src="https://img.shields.io/badge/Join%20our%20Discord-5865F2?style=flat&logo=discord&logoColor=white" alt="加入我们的 Discord"></a>
<a href="https://x.com/PiebaldAI"><img src="https://img.shields.io/badge/Follow%20%40PiebaldAI-000000?style=flat&logo=x&logoColor=white" alt="X"></a>

<sub>[**向下滚动查看 Claude Code 的系统提示。**](#claude-code-system-prompts) :point_down:</sub>

</div>
</div>

<div align="left">
<a href="https://piebald.ai">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://piebald.ai/screenshot-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="https://piebald.ai/screenshot-light.png">
  <img alt="hero" width="800" src="https://piebald.ai/screenshot-light.png">
</picture>
</a>
</div>

# Claude Code 系统提示

[![Mentioned in Awesome Claude Code](https://awesome.re/mentioned-badge.svg)](https://github.com/hesreallyhim/awesome-claude-code)

> [!important]
> **新增（2026 年 1 月 23 日）：我们已将 Claude Code 的约 40 个系统提醒全部添加到此列表中——参见 [系统提醒](#system-reminders)。**

本仓库包含截至 **[Claude Code v2.1.76](https://www.npmjs.com/package/@anthropic-ai/claude-code/v/2.1.76)（2026 年 3 月 13 日）**的所有 Claude Code 各种系统提示及其关联 token 数量的最新列表。它还包含自 v2.0.14 以来 126 个版本的系统提示的 [**CHANGELOG.md**](./CHANGELOG.md)。由 [<img src="https://github.com/Piebald-AI/piebald/raw/main/assets/logo.svg" width="15"> **Piebald.**](https://piebald.ai/) 团队维护

**本仓库会在每次 Claude Code 发布后的几分钟内更新。查看 [更新日志](./CHANGELOG.md)，并在 X 上关注 [@PiebaldAI](https://x.com/PiebaldAI) 以获取每个版本中系统提示更改的摘要。**

> [!note]
> ⭐ **Star** 本仓库以获取新 Claude Code 版本的通知。对于每个新的 Claude Code 版本，我们会在 GitHub 上创建一个 release，这将通知所有 star 了本仓库的用户。

---

为什么有多个"系统提示"？

**Claude Code 的系统提示不仅仅只有一个字符串。**

相反，它包括：
- 根据环境和各种配置有条件地添加的大部分内容。
- 内置工具的描述，如 `Write`、`Bash` 和 `TodoWrite`，其中一些相当大。
- 内置代理（如 Explore 和 Plan）的独立系统提示。
- 众多 AI 驱动的实用函数，如对话压缩、`CLAUDE.md` 生成、会话标题生成等，每个都有自己的系统提示。

结果是——110+ 个字符串，在一个非常大型压缩的 JS 文件中不断变化和移动。

> [!TIP]
> 想要在您自己的 Claude Code 安装中**修改系统提示的特定部分**？**使用 [tweakcc](https://github.com/Piebald-AI/tweakcc)。** 它可以——
> - 让您以 markdown 文件的形式自定义系统提示的各个部分，然后
> - 使用它们修补基于 npm 或原生（二进制）的 Claude Code 安装，并且
> - 为当您和 Anthropic 对同一提示文件有冲突修改时提供差异和冲突管理。

## 提取

本仓库包含使用脚本从最新 npm 版本的 Claude Code 中提取的系统提示。由于它们是直接从 Claude Code 的编译源代码中提取的，因此保证与 Claude Code 使用的完全相同。如果您使用 [tweakcc](https://github.com/Piebald-AI/tweakcc) 来自定义系统提示，它的工作方式类似——它修补本地安装中的相同字符串，这些字符串与本仓库中提取的字符串相同。

## 提示

请注意，某些提示包含插值内容，如内置工具名称引用、可用子代理列表以及各种其他上下文相关变量，因此特定 Claude Code 会话中的实际计数会略有不同——可能不会超过 ±20 tokens。

### Agent 提示

子代理和实用程序。

#### 子代理

- [Agent 提示：Explore 优势和行为指南](./system-prompts/agent-prompt-explore-strengths-and-guidelines.md) (**185** tks) - 定义代码库探索子代理的优势和行为指南，强调搜索策略、彻底性并避免不必要的文件创建。
- [Agent 提示：Explore](./system-prompts/agent-prompt-explore.md) (**517** tks) - Explore 子代理的系统提示。
- [Agent 提示：Plan 模式（增强）](./system-prompts/agent-prompt-plan-mode-enhanced.md) (**685** tks) - Plan 子代理的增强提示。

#### 创建助手

- [Agent 提示：Agent 创建架构师](./system-prompts/agent-prompt-agent-creation-architect.md) (**1110** tks) - 用于创建具有详细规范的自定义 AI 代理的系统提示。
- [Agent 提示：CLAUDE.md 创建](./system-prompts/agent-prompt-claudemd-creation.md) (**384** tks) - 用于分析代码库和创建 CLAUDE.md 文档文件的系统提示。
- [Agent 提示：状态行设置](./system-prompts/agent-prompt-status-line-setup.md) (**1641** tks) - 用于配置状态行显示的 statusline-setup 代理的系统提示。

#### 斜杠命令

- [Agent 提示：/batch 斜杠命令](./system-prompts/agent-prompt-batch-slash-command.md) (**1136** tks) - 用于在代码库中编排大型可并行更改的指令。
- [Agent 提示：/pr-comments 斜杠命令](./system-prompts/agent-prompt-pr-comments-slash-command.md) (**402** tks) - 用于获取和显示 GitHub PR 评论的系统提示。
- [Agent 提示：/review-pr 斜杠命令](./system-prompts/agent-prompt-review-pr-slash-command.md) (**211** tks) - 用于通过代码分析审查 GitHub pull request 的系统提示。
- [Agent 提示：/security-review 斜杠命令](./system-prompts/agent-prompt-security-review-slash-command.md) (**2607** tks) - 综合安全审查提示，用于分析代码更改，重点关注可利用的漏洞。

#### 实用程序

- [Agent 提示：Agent Hook](./system-prompts/agent-prompt-agent-hook.md) (**133** tks) - 用于"agent hook"的提示。
- [Agent 提示：Bash 命令描述编写器](./system-prompts/agent-prompt-bash-command-description-writer.md) (**207** tks) - 用于为 bash 命令生成清晰、简洁的主动语态命令描述的指令。
- [Agent 提示：Bash 命令前缀检测](./system-prompts/agent-prompt-bash-command-prefix-detection.md) (**823** tks) - 用于检测命令前缀和命令注入的系统提示。
- [Agent 提示：Claude 指南代理](./system-prompts/agent-prompt-claude-guide-agent.md) (**744** tks) - 用于 claude-guide 代理的系统提示，帮助用户有效地理解和使用 Claude Code、Claude Agent SDK 和 Claude API。
- [Agent 提示：Coding 会话标题生成器](./system-prompts/agent-prompt-coding-session-title-generator.md) (**181** tks) - 为编码会话生成标题。
- [Agent 提示：通用后缀（响应格式）](./system-prompts/agent-prompt-common-suffix-response-format.md) (**188** tks) - 将响应格式指令附加到代理提示，根据调用者标志在简洁的子代理报告和详细的独立写入之间切换。
- [Agent 提示：对话摘要](./system-prompts/agent-prompt-conversation-summarization.md) (**956** tks) - 用于创建详细对话摘要的系统提示。
- [Agent 提示：确定要附加的内存文件](./system-prompts/agent-prompt-determine-which-memory-files-to-attach.md) (**218** tks) - 用于确定要为主代理附加哪些内存文件的代理。
- [Agent 提示：Hook 条件评估器](./system-prompts/agent-prompt-hook-condition-evaluator.md) (**78** tks) - 用于评估 Claude Code 中的 hook 条件的系统提示。
- [Agent 提示：提示建议生成器 v2](./system-prompts/agent-prompt-prompt-suggestion-generator-v2.md) (**296** tks) - 用于为 Claude Code 生成提示建议的 v2 指令。
- [Agent 提示：快速 PR 创建](./system-prompts/agent-prompt-quick-pr-creation.md) (**806** tks) - 用于创建具有预填充上下文的 commit 和 pull request 的简化提示。
- [Agent 提示：快速 git commit](./system-prompts/agent-prompt-quick-git-commit.md) (**510** tks) - 用于创建具有预填充上下文的单个 git commit 的简化提示。
- [Agent 提示：最近消息摘要](./system-prompts/agent-prompt-recent-message-summarization.md) (**559** tks) - 用于摘要最近消息的代理提示。
- [Agent 提示：自主代理操作的安全监控器（第一部分）](./system-prompts/agent-prompt-security-monitor-for-autonomous-agent-actions-first-part.md) (**2675** tks) - 指示 Claude 充当安全监控器，根据阻止/允许规则评估自主编码代理操作，以防止提示注入、范围蔓延和意外损坏。
- [Agent 提示：自主代理操作的安全监控器（第二部分）](./system-prompts/agent-prompt-security-monitor-for-autonomous-agent-actions-second-part.md) (**2966** tks) - 定义管理代理可能或不可能执行的工具操作的环境上下文、阻止规则和允许例外。
- [Agent 提示：会话搜索助手](./system-prompts/agent-prompt-session-search-assistant.md) (**439** tks) - 用于会话搜索助手的代理提示，根据用户查询和元数据查找相关会话。
- [Agent 提示：会话内存更新指令](./system-prompts/agent-prompt-session-memory-update-instructions.md) (**756** tks) - 用于在对话期间更新会话内存文件的指令。
- [Agent 提示：会话标题和分支生成](./system-prompts/agent-prompt-session-title-and-branch-generation.md) (**307** tks) - 用于生成简洁的会话标题和 git 分支名称的代理。
- [Agent 提示：更新 Magic Docs](./system-prompts/agent-prompt-update-magic-docs.md) (**718** tks) - 用于 magic-docs 代理的提示。
- [Agent 提示：验证专家](./system-prompts/agent-prompt-verification-specialist.md) (**2453** tks) - 用于验证子代理的系统提示，通过运行构建、测试套件、linter 和对抗性探针来对抗性地测试实现，然后发布 PASS/FAIL/PARTIAL 裁决。
- [Agent 提示：WebFetch 摘要器](./system-prompts/agent-prompt-webfetch-summarizer.md) (**189** tks) - 用于为代理汇总来自 WebFetch 的冗长输出的提示，供主模型使用。
- [Agent 提示：Worker fork 执行](./system-prompts/agent-prompt-worker-fork-execution.md) (**370** tks) - 用于分叉的 worker 子代理的系统提示，该代理直接执行指令而不生成进一步的子代理，然后报告结构化结果。

### 数据

嵌入在 Claude Code 中的各种模板文件的内容。

- [数据：Agent SDK 模式 — Python](./system-prompts/data-agent-sdk-patterns-python.md) (**2495** tks) - Python Agent SDK 模式，包括自定义工具、hooks、子代理、MCP 集成和会话恢复。
- [数据：Agent SDK 模式 — TypeScript](./system-prompts/data-agent-sdk-patterns-typescript.md) (**1250** tks) - TypeScript Agent SDK 模式，包括基本代理、hooks、子代理和 MCP 集成。
- [数据：Agent SDK 参考 — Python](./system-prompts/data-agent-sdk-reference-python.md) (**3104** tks) - Python Agent SDK 参考，包括安装、快速入门、通过 MCP 的自定义工具和 hooks。
- [数据：Agent SDK 参考 — TypeScript](./system-prompts/data-agent-sdk-reference-typescript.md) (**2921** tks) - TypeScript Agent SDK 参考，包括安装、快速入门、自定义工具和 hooks。
- [数据：Claude API 参考 — C#](./system-prompts/data-claude-api-reference-c.md) (**4703** tks) - C# SDK 参考，包括安装、客户端初始化、基本请求、流和工具使用。
- [数据：Claude API 参考 — Go](./system-prompts/data-claude-api-reference-go.md) (**4341** tks) - Go SDK 参考。
- [数据：Claude API 参考 — Java](./system-prompts/data-claude-api-reference-java.md) (**4356** tks) - Java SDK 参考，包括安装、客户端初始化、基本请求、流和 beta 工具使用。
- [数据：Claude API 参考 — PHP](./system-prompts/data-claude-api-reference-php.md) (**2381** tks) - PHP SDK 参考。
- [数据：Claude API 参考 — Python](./system-prompts/data-claude-api-reference-python.md) (**3518** tks) - Python SDK 参考，包括安装、客户端初始化、基本请求、思考和多轮对话。
- [数据：Claude API 参考 — Ruby](./system-prompts/data-claude-api-reference-ruby.md) (**696** tks) - Ruby SDK 参考，包括安装、客户端初始化、基本请求、流和 beta 工具运行器。
- [数据：Claude API 参考 — TypeScript](./system-prompts/data-claude-api-reference-typescript.md) (**2837** tks) - TypeScript SDK 参考，包括安装、客户端初始化、基本请求、思考和多轮对话。
- [数据：Claude API 参考 — cURL](./system-prompts/data-claude-api-reference-curl.md) (**1996** tks) - 用于与 cURL 或原始 HTTP 一起使用的 Claude API 的原始 API 参考。
- [数据：Claude 模型目录](./system-prompts/data-claude-model-catalog.md) (**1558** tks) - 当前和传统 Claude 模型的目录，包含确切的模型 ID、别名、上下文窗口和定价。
- [数据：Files API 参考 — Python](./system-prompts/data-files-api-reference-python.md) (**1334** tks) - Python Files API 参考，包括文件上传、列表、删除和在消息中的使用。
- [数据：Files API 参考 — TypeScript](./system-prompts/data-files-api-reference-typescript.md) (**797** tks) - TypeScript Files API 参考，包括文件上传、列表、删除和在消息中的使用。
- [数据：用于 @claude 提及的 GitHub Actions 工作流](./system-prompts/data-github-actions-workflow-for-claude-mentions.md) (**527** tks) - 用于通过 @claude 提及触发 Claude Code 的 GitHub Actions 工作流模板。
- [数据：GitHub App 安装 PR 描述](./system-prompts/data-github-app-installation-pr-description.md) (**424** tks) - 安装 Claude Code GitHub App 集成时的 PR 描述模板。
- [数据：HTTP 错误代码参考](./system-prompts/data-http-error-codes-reference.md) (**1922** tks) - Claude API 返回的 HTTP 错误代码参考，包含常见原因和处理策略。
- [数据：实时文档源](./system-prompts/data-live-documentation-sources.md) (**2336** tks) - 用于从官方源获取当前 Claude API 和 Agent SDK 文档的 WebFetch URL。
- [数据：Message Batches API 参考 — Python](./system-prompts/data-message-batches-api-reference-python.md) (**1544** tks) - Python Batches API 参考，包括批量创建、状态轮询和以 50% 成本检索结果。
- [数据：会话内存模板](./system-prompts/data-session-memory-template.md) (**292** tks) - 会话内存 `summary.md` 文件的模板结构。
- [数据：流式传输参考 — Python](./system-prompts/data-streaming-reference-python.md) (**1528** tks) - Python 流式传输参考，包括同步/异步流式传输和处理不同内容类型。
- [数据：流式传输参考 — TypeScript](./system-prompts/data-streaming-reference-typescript.md) (**1703** tks) - TypeScript 流式传输参考，包括基本流式传输和处理不同内容类型。
- [数据：工具使用概念](./system-prompts/data-tool-use-concepts.md) (**3932** tks) - 使用 Claude API 的工具使用的概念基础，包括工具定义、工具选择和最佳实践。
- [数据：工具使用参考 — Python](./system-prompts/data-tool-use-reference-python.md) (**5106** tks) - Python 工具使用参考，包括工具运行器、手动代理循环、代码执行和结构化输出。
- [数据：工具使用参考 — TypeScript](./system-prompts/data-tool-use-reference-typescript.md) (**5033** tks) - TypeScript 工具使用参考，包括工具运行器、手动代理循环、代码执行和结构化输出。

### 系统提示

主系统提示的部分。

- [系统提示：Agent 摘要生成](./system-prompts/system-prompt-agent-summary-generation.md) (**178** tks) - 用于"Agent 摘要"生成的系统提示。
- [系统提示：Agent 内存指令](./system-prompts/system-prompt-agent-memory-instructions.md) (**337** tks) - 用于在代理系统提示中包含内存更新指导的指令。
- [系统提示：Agent 线程注释](./system-prompts/system-prompt-agent-thread-notes.md) (**216** tks) - 代理线程的行为指南，涵盖绝对路径、响应格式、避免表情符号和工具调用标点符号。
- [系统提示：完整压缩提示的分析指令（完整对话）](./system-prompts/system-prompt-analysis-instructions-for-full-compact-prompt-full-conversation.md) (**182** tks) - 分析指令的系统提示。压缩指令的一部分。完整对话。
- [系统提示：完整压缩提示的分析指令（精简版和通过功能标志）](./system-prompts/system-prompt-analysis-instructions-for-full-compact-prompt-minimal-and-via-feature-flag.md) (**157** tks) - 分析指令的系统提示。压缩指令的一部分。精简版本 - 实验性。
- [系统提示：完整压缩提示的分析指令（最近的消息）](./system-prompts/system-prompt-analysis-instructions-for-full-compact-prompt-recent-messages.md) (**178** tks) - 分析指令的系统提示。压缩指令的一部分。仅最近的消息。
- [系统提示：自动模式](./system-prompts/system-prompt-auto-mode.md) (**188** tks) - 连续任务执行，类似于后台代理。
- [系统提示：审查对恶意活动的协助](./system-prompts/system-prompt-censoring-assistance-with-malicious-activities.md) (**98** tks) - 用于协助授权的安全测试、防御性安全、CTF 挑战和教育上下文同时审查恶意活动请求的指南。
- [系统提示：Chrome 浏览器 MCP 工具](./system-prompts/system-prompt-chrome-browser-mcp-tools.md) (**156** tks) - 用于在使用前通过 MCPSearch 加载 Chrome 浏览器 MCP 工具的指令。
- [系统提示：Claude 在 Chrome 浏览器自动化中](./system-prompts/system-prompt-claude-in-chrome-browser-automation.md) (**759** tks) - 用于有效使用 Claude 在 Chrome 浏览器自动化工具中的指令。
- [系统提示：上下文压缩摘要](./system-prompts/system-prompt-context-compaction-summary.md) (**278** tks) - 用于上下文压缩摘要的提示（用于 SDK）。
- [系统提示：内存指令的描述部分](./system-prompts/system-prompt-description-part-of-memory-instructions.md) (**148** tks) - 用于描述内存是_什么_的字段。指导 Claude 如何创建记忆的更大工作的一部分。
- [系统提示：执行任务（雄心勃勃的任务）](./system-prompts/system-prompt-doing-tasks-ambitious-tasks.md) (**47** tks) - 允许用户完成雄心勃勃的任务；在范围上听从用户的判断。
- [系统提示：执行任务（避免过度工程）](./system-prompts/system-prompt-doing-tasks-avoid-over-engineering.md) (**30** tks) - 仅进行直接请求或明确必要的更改。
- [系统提示：执行任务（阻止的方法）](./system-prompts/system-prompt-doing-tasks-blocked-approach.md) (**90** tks) - 在被阻止时考虑替代方案，而不是强行攻击。
- [系统提示：执行任务（帮助和反馈）](./system-prompts/system-prompt-doing-tasks-help-and-feedback.md) (**24** tks) - 如何通知用户帮助和反馈渠道。
- [系统提示：执行任务（最小化文件创建）](./system-prompts/system-prompt-doing-tasks-minimize-file-creation.md) (**47** tks) - 优先编辑现有文件而不是创建新文件。
- [系统提示：执行任务（无兼容性黑客）](./system-prompts/system-prompt-doing-tasks-no-compatibility-hacks.md) (**52** tks) - 完全删除未使用的代码，而不是添加兼容性垫片。
- [系统提示：执行任务（无过早抽象）](./system-prompts/system-prompt-doing-tasks-no-premature-abstractions.md) (**60** tks) - 不要为一次性操作或假设要求创建抽象。
- [系统提示：执行任务（无时间估算）](./system-prompts/system-prompt-doing-tasks-no-time-estimates.md) (**47** tks) - 避免给出时间估算或预测。
- [系统提示：执行任务（无不必要的添加）](./system-prompts/system-prompt-doing-tasks-no-unnecessary-additions.md) (**78** tks) - 不要添加超出要求的功能、重构或改进。
- [系统提示：执行任务（无不必要的错误处理）](./system-prompts/system-prompt-doing-tasks-no-unnecessary-error-handling.md) (**64** tks) - 不要为不可能的场景添加错误处理；仅在边界处验证。
- [系统提示：执行任务（修改前先读取）](./system-prompts/system-prompt-doing-tasks-read-before-modifying.md) (**46** tks) - 在建议修改之前读取并理解现有代码。
- [系统提示：执行任务（安全）](./system-prompts/system-prompt-doing-tasks-security.md) (**67** tks) - 避免引入安全漏洞，如注入、XSS 等。
- [系统提示：执行任务（软件工程重点）](./system-prompts/system-prompt-doing-tasks-software-engineering-focus.md) (**104** tks) - 用户主要请求软件工程任务；在该上下文中解释指令。
- [系统提示：谨慎执行操作](./system-prompts/system-prompt-executing-actions-with-care.md) (**541** tks) - 用于谨慎执行操作的指令。
- [系统提示：Fork 使用指南](./system-prompts/system-prompt-fork-usage-guidelines.md) (**339** tks) - 关于何时分叉子代理的指令，以及禁止在中间读取 fork 输出或伪造 fork 结果的规则。
- [系统提示：Git 状态](./system-prompts/system-prompt-git-status.md) (**97** tks) - 用于在对话开始时显示当前 git 状态的系统提示。
- [系统提示：Hooks 配置](./system-prompts/system-prompt-hooks-configuration.md) (**1482** tks) - 用于 hooks 配置的系统提示。用于上述 Claude Code 配置技能。
- [系统提示：如何使用 SendUserMessage 工具](./system-prompts/system-prompt-how-to-use-the-sendusermessage-tool.md) (**283** tks) - 用于使用 SendUserMessage 工具的指令。
- [系统提示：一目了然的见解摘要](./system-prompts/system-prompt-insights-at-a-glance-summary.md) (**569** tks) - 为见解报告生成简洁的 4 部分摘要（有效的内容、阻碍、快速胜利、雄心勃勃的工作流程）。
- [系统提示：见解摩擦分析](./system-prompts/system-prompt-insights-friction-analysis.md) (**139** tks) - 分析聚合的使用数据以识别摩擦模式并分类重复出现的问题。
- [系统提示：未来的见解](./system-prompts/system-prompt-insights-on-the-horizon.md) (**148** tks) - 识别雄心勃勃的未来工作流程和自主 AI 辅助开发的机会。
- [系统提示：见解会话方面提取](./system-prompts/system-prompt-insights-session-facets-extraction.md) (**310** tks) - 从单个 Claude Code 会话记录中提取结构化方面（目标类别、满意度、摩擦）。
- [系统提示：见解建议](./system-prompts/system-prompt-insights-suggestions.md) (**748** tks) - 生成可操作的建议，包括 CLAUDE.md 添加、要尝试的功能和使用模式。
- [系统提示：学习模式（见解）](./system-prompts/system-prompt-learning-mode-insights.md) (**142** tks) - 当学习模式处于活动状态时提供教育见解的指令。
- [系统提示：学习模式](./system-prompts/system-prompt-learning-mode.md) (**1042** tks) - 带有人类协作指令的学习模式的主系统提示。
- [系统提示：内存系统（私人反馈）](./system-prompts/system-prompt-memory-system-private-feedback.md) (**112** tks) - 描述私人反馈内存类型，用于存储用户指导和更正，指令在保存前检查与团队反馈的矛盾。
- [系统提示：使用 sleep 命令的六条规则之一](./system-prompts/system-prompt-one-of-six-rules-for-using-sleep-command.md) (**23** tks) - 使用 sleep 命令的六条规则之一。
- [系统提示：选项预览器](./system-prompts/system-prompt-option-previewer.md) (**151** tks) - 用于以并排布局预览 UI 选项的系统提示。
- [系统提示：输出效率](./system-prompts/system-prompt-output-efficiency.md) (**177** tks) - 指示 Claude 在文本输出中简洁直接，以答案为主导而非推理，并将响应限制在基本信息内。
- [系统提示：并行工具调用说明（"工具使用策略"的一部分）](./system-prompts/system-prompt-parallel-tool-call-note-part-of-tool-usage-policy.md) (**102** tks) - 用于告诉 Claude 使用并行工具调用的系统提示。
- [系统提示：Plan 模式的第四阶段](./system-prompts/system-prompt-phase-four-of-plan-mode.md) (**142** tks) - Plan 模式的第四阶段。
- [系统提示：草稿目录](./system-prompts/system-prompt-scratchpad-directory.md) (**170** tks) - 用于使用专用草稿目录存储临时文件的指令。
- [系统提示：Skillify 当前会话](./system-prompts/system-prompt-skillify-current-session.md) (**1882** tks) - 用于将当前会话转换为技能的系统提示。
- [系统提示：Subagent 委托示例](./system-prompts/system-prompt-subagent-delegation-examples.md) (**588** tks) - 提供示例交互，展示协调代理应如何向子代理委派任务、处理等待状态和报告结果。
- [系统提示：系统部分](./system-prompts/system-prompt-system-section.md) (**156** tks) - 主系统提示的系统部分。
- [系统提示：团队内存内容显示](./system-prompts/system-prompt-team-memory-content-display.md) (**52** tks) - 使用路径和内容呈现共享团队内存文件内容，以便注入到对话上下文中。
- [系统提示：队友通信](./system-prompts/system-prompt-teammate-communication.md) (**130** tks) - 用于 swarm 中队友通信的系统提示。
- [系统提示：语气和风格（代码引用）](./system-prompts/system-prompt-tone-and-style-code-references.md) (**39** tks) - 引用代码时包含 file_path:line_number 的指令。
- [系统提示：语气和风格（简洁输出 — 详细）](./system-prompts/system-prompt-tone-and-style-concise-output-detailed.md) (**89** tks) - 用于简洁、精炼输出而无填充或内心独白的指令。
- [系统提示：语气和风格（简洁输出 — 简短）](./system-prompts/system-prompt-tone-and-style-concise-output-short.md) (**16** tks) - 用于简短简洁响应的指令。
- [系统提示：工具使用摘要生成](./system-prompts/system-prompt-tool-use-summary-generation.md) (**171** tks) - 用于生成工具使用摘要的提示。
- [系统提示：工具执行被拒绝](./system-prompts/system-prompt-tool-execution-denied.md) (**144** tks) - 当工具执行被拒绝时的系统提示。
- [系统提示：工具使用（创建文件）](./system-prompts/system-prompt-tool-use-create-files.md) (**30** tks) - 优先使用 Write 工具而不是 cat heredoc 或 echo 重定向。
- [系统提示：工具使用（委派探索）](./system-prompts/system-prompt-tool-use-delegate-exploration.md) (**95** tks) - 使用 Task 工具进行更广泛的代码库探索和深度研究。
- [系统提示：工具使用（直接搜索）](./system-prompts/system-prompt-tool-use-direct-search.md) (**39** tks) - 直接使用 Glob/Grep 进行简单、定向的搜索。
- [系统提示：工具使用（编辑文件）](./system-prompts/system-prompt-tool-use-edit-files.md) (**26** tks) - 优先使用 Edit 工具而不是 sed/awk。
- [系统提示：工具使用（读取文件）](./system-prompts/system-prompt-tool-use-read-files.md) (**29** tks) - 优先使用 Read 工具而不是 cat/head/tail/sed。
- [系统提示：工具使用（保留 Bash）](./system-prompts/system-prompt-tool-use-reserve-bash.md) (**75** tks) - 专门为 Bash 工具保留系统命令和终端操作。
- [系统提示：工具使用（搜索内容）](./system-prompts/system-prompt-tool-use-search-content.md) (**30** tks) - 优先使用 Grep 工具而不是 grep 或 rg。
- [系统提示：工具使用（搜索文件）](./system-prompts/system-prompt-tool-use-search-files.md) (**26** tks) - 优先使用 Glob 工具而不是 find 或 ls。
- [系统提示：工具使用（技能调用）](./system-prompts/system-prompt-tool-use-skill-invocation.md) (**102** tks) - 斜杠命令通过 Skill 工具调用用户可调用的技能。
- [系统提示：工具使用（subagent 指导）](./system-prompts/system-prompt-tool-use-subagent-guidance.md) (**103** tks) - 关于何时以及如何有效使用子代理的指导。
- [系统提示：工具使用（任务管理）](./system-prompts/system-prompt-tool-use-task-management.md) (**73** tks) - 使用 TodoWrite 分解和跟踪工作进度。
- [系统提示：Worker 指令](./system-prompts/system-prompt-worker-instructions.md) (**272** tks) - Worker 在实施更改时应遵循的指令。
- [系统提示：编写 subagent 提示](./system-prompts/system-prompt-writing-subagent-prompts.md) (**365** tks) - 用于在向子代理委派任务时编写有效提示的指南，涵盖上下文继承与全新子代理场景。

### 系统提醒

大型系统提醒的文本。

- [系统提醒：/btw 侧面问题](./system-prompts/system-reminder-btw-side-question.md) (**244** tks) - 用于 /btw 斜杠命令没有工具的侧面问题的系统提醒。
- [系统提醒：Agent 提及](./system-prompts/system-reminder-agent-mention.md) (**45** tks) - 用户想要调用代理的通知。
- [系统提醒：压缩文件引用](./system-prompts/system-reminder-compact-file-reference.md) (**57** tks) - 对对话摘要前读取的文件的引用。
- [系统提醒：已退出 Plan 模式](./system-prompts/system-reminder-exited-plan-mode.md) (**73** tks) - 退出 Plan 模式时的通知。
- [系统提醒：文件存在但为空](./system-prompts/system-reminder-file-exists-but-empty.md) (**27** tks) - 读取空文件时的警告。
- [系统提醒：文件被用户或 linter 修改](./system-prompts/system-reminder-file-modified-by-user-or-linter.md) (**97** tks) - 文件被外部修改的通知。
- [系统提醒：文件在 IDE 中打开](./system-prompts/system-reminder-file-opened-in-ide.md) (**37** tks) - 用户在 IDE 中打开文件的通知。
- [系统提醒：文件短于偏移量](./system-prompts/system-reminder-file-shorter-than-offset.md) (**59** tks) - 文件读取偏移量超过文件长度时的警告。
- [系统提醒：文件被截断](./system-prompts/system-reminder-file-truncated.md) (**74** tks) - 文件因大小而被截断的通知。
- [系统提醒：Hook 附加上下文](./system-prompts/system-reminder-hook-additional-context.md) (**35** tks) - 来自 hook 的附加上下文。
- [系统提醒：Hook 阻止错误](./system-prompts/system-reminder-hook-blocking-error.md) (**52** tks) - 来自阻止 hook 命令的错误。
- [系统提醒：Hook 停止继续前缀](./system-prompts/system-reminder-hook-stopped-continuation-prefix.md) (**12** tks) - Hook 停止继续消息的前缀。
- [系统提醒：Hook 停止继续](./system-prompts/system-reminder-hook-stopped-continuation.md) (**30** tks) - Hook 停止继续时的消息。
- [系统提醒：Hook 成功](./system-prompts/system-reminder-hook-success.md) (**29** tks) - 来自 hook 的成功消息。
- [系统提醒：调用的技能](./system-prompts/system-reminder-invoked-skills.md) (**33** tks) - 此会话中调用的技能列表。
- [系统提醒：IDE 中选中的行](./system-prompts/system-reminder-lines-selected-in-ide.md) (**66** tks) - 关于用户在 IDE 中选中的行的通知。
- [系统提醒：MCP 资源无内容](./system-prompts/system-reminder-mcp-resource-no-content.md) (**41** tks) - 当 MCP 资源没有内容时显示。
- [系统提醒：MCP 资源无可显示内容](./system-prompts/system-reminder-mcp-resource-no-displayable-content.md) (**43** tks) - 当 MCP 资源没有可显示内容时显示。
- [系统提醒：Read 工具调用后的恶意软件分析](./system-prompts/system-reminder-malware-analysis-after-read-tool-call.md) (**87** tks) - 用于在不改进或增强的情况下分析恶意软件的指令。
- [系统提醒：内存文件内容](./system-prompts/system-reminder-memory-file-contents.md) (**38** tks) - 按路径的内存文件内容。
- [系统提醒：嵌套内存内容](./system-prompts/system-reminder-nested-memory-contents.md) (**33** tks) - 嵌套内存文件的内容。
- [系统提醒：检测到新的诊断](./system-prompts/system-reminder-new-diagnostics-detected.md) (**35** tks) - 关于新诊断问题的通知。
- [系统提醒：输出样式处于活动状态](./system-prompts/system-reminder-output-style-active.md) (**32** tks) - 输出样式处于活动状态的通知。
- [系统提醒：Plan 文件引用](./system-prompts/system-reminder-plan-file-reference.md) (**62** tks) - 对现有计划文件的引用。
- [系统提醒：Plan 模式处于活动状态（5 阶段）](./system-prompts/system-reminder-plan-mode-is-active-5-phase.md) (**1297** tks) - 增强的计划模式系统提醒，具有并行探索和多代理规划。
- [系统提醒：Plan 模式处于活动状态（迭代）](./system-prompts/system-reminder-plan-mode-is-active-iterative.md) (**919** tks) - 用于主代理的迭代计划模式系统提醒，具有用户访谈工作流程。
- [系统提醒：Plan 模式处于活动状态（subagent）](./system-prompts/system-reminder-plan-mode-is-active-subagent.md) (**307** tks) - 用于子代理的简化计划模式系统提醒。
- [系统提醒：Plan 模式重新进入](./system-prompts/system-reminder-plan-mode-re-entry.md) (**236** tks) - 当用户之前通过 shift+tab 或批准 Claude 的计划退出 Plan 模式后再次进入时发送的系统提醒。
- [系统提醒：会话继续](./system-prompts/system-reminder-session-continuation.md) (**37** tks) - 会话从另一台机器继续的通知。
- [系统提醒：任务状态](./system-prompts/system-reminder-task-status.md) (**18** tks) - 带有 TaskOutput 工具引用的任务状态。
- [系统提醒：任务工具提醒](./system-prompts/system-reminder-task-tools-reminder.md) (**123** tks) - 使用任务跟踪工具的提醒。
- [系统提醒：团队协调](./system-prompts/system-reminder-team-coordination.md) (**250** tks) - 用于团队协调的系统提醒。
- [系统提醒：团队关闭](./system-prompts/system-reminder-team-shutdown.md) (**136** tks) - 用于团队关闭的系统提醒。
- [系统提醒：TodoWrite 提醒](./system-prompts/system-reminder-todowrite-reminder.md) (**98** tks) - 使用 TodoWrite 工具进行任务跟踪的提醒。
- [系统提醒：Token 使用情况](./system-prompts/system-reminder-token-usage.md) (**39** tks) - 当前 token 使用情况统计。
- [系统提醒：USD 预算](./system-prompts/system-reminder-usd-budget.md) (**42** tks) - 当前 USD 预算统计。
- [系统提醒：验证计划提醒](./system-prompts/system-reminder-verify-plan-reminder.md) (**47** tks) - 验证已完成计划的提醒。

### 内置工具描述

- [工具描述：AskUserQuestion](./system-prompts/tool-description-askuserquestion.md) (**287** tks) - 用于向用户提问的工具描述。
- [工具描述：Computer](./system-prompts/tool-description-computer.md) (**161** tks) - Chrome 浏览器计算机自动化工具的主要描述。
- [工具描述：CronCreate](./system-prompts/tool-description-croncreate.md) (**738** tks) - 描述 CronCreate 工具，用于将一次性或定期基于 cron 的作业排入队列，包含抖动和非分钟调度指导。
- [工具描述：Edit](./system-prompts/tool-description-edit.md) (**246** tks) - 用于在文件中执行精确字符串替换的工具。
- [工具描述：EnterPlanMode](./system-prompts/tool-description-enterplanmode.md) (**878** tks) - 用于进入 Plan 模式以探索和设计实现方法的工具描述。
- [工具描述：EnterWorktree](./system-prompts/tool-description-enterworktree.md) (**359** tks) - EnterWorktree 工具的工具描述。
- [工具描述：ExitPlanMode](./system-prompts/tool-description-exitplanmode.md) (**417** tks) - ExitPlanMode 工具的描述，显示计划对话框供用户批准。
- [工具描述：ExitWorktree](./system-prompts/tool-description-exitworktree.md) (**527** tks) - 大致上是 ExitWorktree 的反向。
- [工具描述：Glob](./system-prompts/tool-description-glob.md) (**122** tks) - 用于文件模式匹配和按名称搜索的工具描述。
- [工具描述：Grep](./system-prompts/tool-description-grep.md) (**300** tks) - 用于使用 ripgrep 进行内容搜索的工具描述。
- [工具描述：LSP](./system-prompts/tool-description-lsp.md) (**255** tks) - LSP 工具的描述。
- [工具描述：NotebookEdit](./system-prompts/tool-description-notebookedit.md) (**121** tks) - 用于编辑 Jupyter notebook 单元格的工具描述。
- [工具描述：ReadFile](./system-prompts/tool-description-readfile.md) (**440** tks) - 用于读取文件的工具描述。
- [工具描述：SendMessageTool](./system-prompts/tool-description-sendmessagetool.md) (**1205** tks) - SendMessageTool 的代理团队版本。
- [工具描述：Skill](./system-prompts/tool-description-skill.md) (**326** tks) - 用于在主对话中执行技能的工具描述。
- [工具描述：Sleep](./system-prompts/tool-description-sleep.md) (**154** tks) - 用于等待/睡眠的工具，具有在用户输入时提前唤醒的功能。
- [工具描述：TaskCreate](./system-prompts/tool-description-taskcreate.md) (**528** tks) - TaskCreate 工具的工具描述。
- [工具描述：TeamDelete](./system-prompts/tool-description-teamdelete.md) (**154** tks) - TeamDelete 工具的工具描述。
- [工具描述：TeammateTool](./system-prompts/tool-description-teammatetool.md) (**1645** tks) - 用于在 swarm 中管理团队和协调队友的工具。
- [工具描述：TodoWrite](./system-prompts/tool-description-todowrite.md) (**2161** tks) - 用于创建和管理任务列表的工具描述。
- [工具描述：WebFetch](./system-prompts/tool-description-webfetch.md) (**297** tks) - 用于 web 获取功能的工具描述。
- [工具描述：WebSearch](./system-prompts/tool-description-websearch.md) (**321** tks) - 用于 web 搜索功能的工具描述。
- [工具描述：Write](./system-prompts/tool-description-write.md) (**129** tks) - 用于将文件写入本地文件系统的工具。

**某些工具描述的附加说明**

- [工具描述：Agent（使用说明）](./system-prompts/tool-description-agent-usage-notes.md) (**931** tks) - Task/Agent 工具的使用说明和指令，包括关于启动子代理、后台执行、恢复和工作隔离的指导。
- [工具描述：Agent（何时启动子代理）](./system-prompts/tool-description-agent-when-to-launch-subagents.md) (**186** tks) - 描述_何时_使用 Agent 工具——用于启动专门的子代理子进程以自主处理复杂的多步骤任务。
- [工具描述：AskUserQuestion（预览字段）](./system-prompts/tool-description-askuserquestion-preview-field.md) (**134** tks) - 用于在单选问题选项上使用 HTML 预览字段以显示视觉内容（如 UI 模型、代码片段和图表）的指令。
- [工具描述：Bash（Git commit 和 PR 创建指令）](./system-prompts/tool-description-bash-git-commit-and-pr-creation-instructions.md) (**1558** tks) - 用于创建 git commits 和 GitHub pull requests 的指令。
- [工具描述：Bash（替代方案 — 通信）](./system-prompts/tool-description-bash-alternative-communication.md) (**18** tks) - Bash 工具替代方案：直接输出文本而不是 echo/printf。
- [工具描述：Bash（替代方案 — 内容搜索）](./system-prompts/tool-description-bash-alternative-content-search.md) (**27** tks) - Bash 工具替代方案：使用 Grep 进行内容搜索而不是 grep/rg。
- [工具描述：Bash（替代方案 — 编辑文件）](./system-prompts/tool-description-bash-alternative-edit-files.md) (**27** tks) - Bash 工具替代方案：使用 Edit 进行文件编辑而不是 sed/awk。
- [工具描述：Bash（替代方案 — 文件搜索）](./system-prompts/tool-description-bash-alternative-file-search.md) (**26** tks) - Bash 工具替代方案：使用 Glob 进行文件搜索而不是 find/ls。
- [工具描述：Bash（替代方案 — 读取文件）](./system-prompts/tool-description-bash-alternative-read-files.md) (**27** tks) - Bash 工具替代方案：使用 Read 进行文件读取而不是 cat/head/tail。
- [工具描述：Bash（替代方案 — 写入文件）](./system-prompts/tool-description-bash-alternative-write-files.md) (**29** tks) - Bash 工具替代方案：使用 Write 进行文件写入而不是 echo/cat。
- [工具描述：Bash（内置工具说明）](./system-prompts/tool-description-bash-built-in-tools-note.md) (**53** tks) - 注意内置工具比 Bash 等效工具提供更好的用户体验。
- [工具描述：Bash（命令描述）](./system-prompts/tool-description-bash-command-description.md) (**71** tks) - Bash 工具指令：编写清晰的命令描述。
- [工具描述：Bash（git — 避免破坏性操作）](./system-prompts/tool-description-bash-git-avoid-destructive-ops.md) (**58** tks) - Bash 工具 git 指令：考虑破坏性操作的更安全替代方案。
- [工具描述：Bash（git — 永不跳过 hooks）](./system-prompts/tool-description-bash-git-never-skip-hooks.md) (**59** tks) - Bash 工具 git 指令：除非用户请求，否则永不跳过 hooks 或绕过签名。
- [工具描述：Bash（git — 优先新提交）](./system-prompts/tool-description-bash-git-prefer-new-commits.md) (**22** tks) - Bash 工具 git 指令：优先新提交而不是修改。
- [工具描述：Bash（维护 cwd）](./system-prompts/tool-description-bash-maintain-cwd.md) (**41** tks) - Bash 工具指令：使用绝对路径并避免 cd。
- [工具描述：Bash（无换行符）](./system-prompts/tool-description-bash-no-newlines.md) (**24** tks) - Bash 工具指令：不要使用换行符分隔命令。
- [工具描述：Bash（概述）](./system-prompts/tool-description-bash-overview.md) (**19** tks) - Bash 工具描述的开头行。
- [工具描述：Bash（并行命令）](./system-prompts/tool-description-bash-parallel-commands.md) (**72** tks) - Bash 工具指令：将独立命令作为并行工具调用运行。
- [工具描述：Bash（优先使用专用工具）](./system-prompts/tool-description-bash-prefer-dedicated-tools.md) (**71** tks) - 警告优先使用专用工具而不是 Bash 进行 find、grep、cat 等操作。
- [工具描述：Bash（引用文件路径）](./system-prompts/tool-description-bash-quote-file-paths.md) (**35** tks) - Bash 工具指令：引用包含空格的文件路径。
- [工具描述：Bash（sandbox — 调整设置）](./system-prompts/tool-description-bash-sandbox-adjust-settings.md) (**26** tks) - 与用户合作在失败时调整 sandbox 设置。
- [工具描述：Bash（sandbox — 默认使用 sandbox）](./system-prompts/tool-description-bash-sandbox-default-to-sandbox.md) (**38** tks) - 默认使用 sandbox；仅在用户要求或有 sandbox 限制证据时绕过。
- [工具描述：Bash（sandbox — 证据列表标题）](./system-prompts/tool-description-bash-sandbox-evidence-list-header.md) (**15** tks) - sandbox 导致的失败证据列表的标题。
- [工具描述：Bash（sandbox — 证据：访问被拒绝）](./system-prompts/tool-description-bash-sandbox-evidence-access-denied.md) (**15** tks) - Sandbox 证据：对允许目录之外的路径访问被拒绝。
- [工具描述：Bash（sandbox — 证据：网络失败）](./system-prompts/tool-description-bash-sandbox-evidence-network-failures.md) (**17** tks) - Sandbox 证据：到非白名单主机的网络连接失败。
- [工具描述：Bash（sandbox — 证据：操作不允许）](./system-prompts/tool-description-bash-sandbox-evidence-operation-not-permitted.md) (**18** tks) - Sandbox 证据：操作不允许错误。
- [工具描述：Bash（sandbox — 证据：unix socket 错误）](./system-prompts/tool-description-bash-sandbox-evidence-unix-socket-errors.md) (**11** tks) - Sandbox 证据：unix socket 连接错误。
- [工具描述：Bash（sandbox — 解释限制）](./system-prompts/tool-description-bash-sandbox-explain-restriction.md) (**36** tks) - 解释哪种 sandbox 限制导致了失败。
- [工具描述：Bash（sandbox — 失败证据条件）](./system-prompts/tool-description-bash-sandbox-failure-evidence-condition.md) (**48** tks) - 条件：命令失败并有 sandbox 限制的证据。
- [工具描述：Bash（sandbox — 强制模式）](./system-prompts/tool-description-bash-sandbox-mandatory-mode.md) (**34** tks) - 策略：所有命令必须在 sandbox 模式下运行。
- [工具描述：Bash（sandbox — 无例外）](./system-prompts/tool-description-bash-sandbox-no-exceptions.md) (**17** tks) - 命令在任何情况下都不能在 sandbox 之外运行。
- [工具描述：Bash（sandbox — 无敏感路径）](./system-prompts/tool-description-bash-sandbox-no-sensitive-paths.md) (**36** tks) - 不要建议将敏感路径添加到 sandbox 允许列表。
- [工具描述：Bash（sandbox — 每个命令）](./system-prompts/tool-description-bash-sandbox-per-command.md) (**52** tks) - 单独处理每个命令；默认为未来命令使用 sandbox。
- [工具描述：Bash（sandbox — 响应标题）](./system-prompts/tool-description-bash-sandbox-response-header.md) (**17** tks) - 关于如何响应看到 sandbox 导致的失败的标题。
- [工具描述：Bash（sandbox — 在没有 sandbox 的情况下重试）](./system-prompts/tool-description-bash-sandbox-retry-without-sandbox.md) (**33** tks) - 在 sandbox 失败时立即使用 dangerouslyDisableSandbox 重试。
- [工具描述：Bash（sandbox — tmpdir）](./system-prompts/tool-description-bash-sandbox-tmpdir.md) (**102** tks) - 在 sandbox 模式下使用 $TMPDIR 存储临时文件。
- [工具描述：Bash（sandbox — 用户权限提示）](./system-prompts/tool-description-bash-sandbox-user-permission-prompt.md) (**14** tks) - 注意禁用 sandbox 将提示用户许可。
- [工具描述：Bash（分号使用）](./system-prompts/tool-description-bash-semicolon-usage.md) (**29** tks) - Bash 工具指令：当顺序顺序重要但失败不重要时使用分号。
- [工具描述：Bash（顺序命令）](./system-prompts/tool-description-bash-sequential-commands.md) (**42** tks) - Bash 工具指令：用 && 链接依赖命令。
- [工具描述：Bash（sleep — 保持简短）](./system-prompts/tool-description-bash-sleep-keep-short.md) (**29** tks) - Bash 工具指令：将 sleep 持续时间保持在 1-5 秒。
- [工具描述：Bash（sleep — 不轮询后台任务）](./system-prompts/tool-description-bash-sleep-no-polling-background-tasks.md) (**37** tks) - Bash 工具指令：不要轮询后台任务，等待通知。
- [工具描述：Bash（sleep — 立即运行）](./system-prompts/tool-description-bash-sleep-run-immediately.md) (**21** tks) - Bash 工具指令：在可以立即运行的命令之间不要 sleep。
- [工具描述：Bash（sleep — 使用检查命令）](./system-prompts/tool-description-bash-sleep-use-check-commands.md) (**34** tks) - Bash 工具指令：在轮询时使用检查命令而不是 sleep。
- [工具描述：Bash（超时）](./system-prompts/tool-description-bash-timeout.md) (**83** tks) - Bash 工具指令：可选超时配置。
- [工具描述：Bash（验证父目录）](./system-prompts/tool-description-bash-verify-parent-directory.md) (**38** tks) - Bash 工具指令：在创建文件之前验证父目录。
- [工具描述：Bash（工作目录）](./system-prompts/tool-description-bash-working-directory.md) (**37** tks) - 关于工作目录持久化和 shell 状态的 Bash 工具说明。
- [工具描述：SendMessageTool（非代理团队）](./system-prompts/tool-description-sendmessagetool-non-agent-teams.md) (**133** tks) - 发送用户将读取的消息，很好地描述了此工具。
- [工具描述：TaskList（队友工作流程）](./system-prompts/tool-description-tasklist-teammate-workflow.md) (**133** tks) - 附加到 TaskList 工具描述的条件部分。
- [工具描述：ToolSearch（第二部分）](./system-prompts/tool-description-toolsearch-second-part.md) (**202** tks) - 工具描述的大部分。
- [工具参数：Computer action](./system-prompts/tool-parameter-computer-action.md) (**251** tks) - Chrome 浏览器计算机工具的操作参数选项。

### 技能

用于专门任务的内置技能提示。

- [技能：/loop 斜杠命令](./system-prompts/skill-loop-slash-command.md) (**984** tks) - 将用户输入解析为间隔和提示，将间隔转换为 cron 表达式，并安排定期任务。
- [技能：/stuck 斜杠命令](./system-prompts/skill-stuck-slash-command.md) (**865** tks) - 诊断冻结或缓慢的 Claude Code 会话。
- [技能：使用 Claude API 构建（参考指南）](./system-prompts/skill-build-with-claude-api-reference-guide.md) (**410** tks) - 用于展示具有快速任务导航的特定语言参考文档的模板。
- [技能：使用 Claude API 构建](./system-prompts/skill-build-with-claude-api.md) (**5144** tks) - 使用 Claude 构建 LLM 驱动应用程序的主要路由指南，包括语言检测、表面选择和架构概述。
- [技能：创建验证器技能](./system-prompts/skill-create-verifier-skills.md) (**2625** tks) - 用于为 Verify 代理创建验证器技能以自动验证代码更改的提示。
- [技能：调试](./system-prompts/skill-debugging.md) (**412** tks) - 用于调试用户在 Claude Code 会话中遇到的问题的指令。
- [技能：简化](./system-prompts/skill-simplify.md) (**822** tks) - 用于简化代码的指令。
- [技能：更新 Claude Code 配置](./system-prompts/skill-update-claude-code-config.md) (**1232** tks) - 用于修改 Claude Code 配置文件（settings.json）的技能。
- [技能：验证专家](./system-prompts/skill-verification-specialist.md) (**2472** tks) - 用于验证代码更改正常工作的技能。
