<!--
name: 'Skill: Build with Claude API'
description: Main routing guide for building LLM-powered applications with Claude, including language detection, surface selection, and architecture overview
ccVersion: 2.1.73
-->
# 使用 Claude 构建 LLM 驱动的应用程序

本技能帮助您使用 Claude 构建 LLM 驱动的应用程序。根据您的需求选择合适的接口，检测项目语言，然后阅读相关的特定语言文档。

## 默认设置

除非用户另有要求：

对于 Claude 模型版本，请使用 {{OPUS_NAME}}，您可以通过精确的模型字符串 `{{OPUS_ID}}` 访问它。请默认对任何稍微复杂的请求使用自适应思考（`thinking: {type: "adaptive"}`）。最后，对于任何可能涉及长输入、长输出或高 `max_tokens` 的请求，请默认使用流式传输 — 这可以防止请求超时。如果不需要处理单个流事件，请使用 SDK 的 `.get_final_message()` / `.finalMessage()` 辅助器来获取完整响应

---

## 语言检测

在阅读代码示例之前，确定用户使用的语言：

1. **查看项目文件**以推断语言：

   - `*.py`、`requirements.txt`、`pyproject.toml`、`setup.py`、`Pipfile` → **Python** — 从 `python/` 阅读
   - `*.ts`、`*.tsx`、`package.json`、`tsconfig.json` → **TypeScript** — 从 `typescript/` 阅读
   - `*.js`、`*.jsx`（没有 `.ts` 文件）→ **TypeScript** — JS 使用相同的 SDK，从 `typescript/` 阅读
   - `*.java`、`pom.xml`、`build.gradle` → **Java** — 从 `java/` 阅读
   - `*.kt`、`*.kts`、`build.gradle.kts` → **Java** — Kotlin 使用 Java SDK，从 `java/` 阅读
   - `*.scala`、`build.sbt` → **Java** — Scala 使用 Java SDK，从 `java/` 阅读
   - `*.go`、`go.mod` → **Go** — 从 `go/` 阅读
   - `*.rb`、`Gemfile` → **Ruby** — 从 `ruby/` 阅读
   - `*.cs`、`*.csproj` → **C#** — 从 `csharp/` 阅读
   - `*.php`、`composer.json` → **PHP** — 从 `php/` 阅读

2. **如果检测到多种语言**（例如，Python 和 TypeScript 文件）：

   - 检查用户当前文件或问题与哪种语言相关
   - 如果仍然模棱两可，询问："我检测到了 Python 和 TypeScript 文件。您使用哪种语言进行 Claude API 集成？"

3. **如果无法推断语言**（空项目、没有源文件或不支持的语言）：

   - 使用 AskUserQuestion 提供选项：Python、TypeScript、Java、Go、Ruby、cURL/raw HTTP、C#、PHP
   - 如果 AskUserQuestion 不可用，默认使用 Python 示例并注意："显示 Python 示例。如果您需要不同的语言，请告诉我。"

4. **如果检测到不支持的语言**（Rust、Swift、C++、Elixir 等）：

   - 建议使用 `curl/` 中的 cURL/raw HTTP 示例，并注意可能存在社区 SDK
   - 提供显示 Python 或 TypeScript 示例作为参考实现

5. **如果用户需要 cURL/raw HTTP 示例**，从 `curl/` 阅读。

### 特定语言的功能支持

| 语言 | 工具运行器 | Agent SDK | 说明 |
| ---------- | ----------- | --------- | ------------------------------------- |
| Python | 是（beta） | 是 | 完全支持 — `@beta_tool` 装饰器 |
| TypeScript | 是（beta） | 是 | 完全支持 — `betaZodTool` + Zod |
| Java | 是（beta） | 否 | 带注解类的 Beta 工具使用 |
| Go | 是（beta） | 否 | `toolrunner` 包中的 `BetaToolRunner` |
| Ruby | 是（beta） | 否 | beta 中的 `BaseTool` + `tool_runner` |
| cURL | N/A | N/A | 原始 HTTP，无 SDK 功能 |
| C# | 否 | 否 | 官方 SDK |
| PHP | 否 | 否 | 官方 SDK |

---

## 我应该使用哪个接口？

> **从简单开始。** 默认使用满足您需求的最简单层。单个 API 调用和工作流处理大多数用例 — 仅当任务真正需要开放式、模型驱动的探索时才使用 agent。

| 用例 | 层 | 推荐接口 | 原因 |
| ----------------------------------------------- | --------------- | ------------------------- | --------------------------------------- |
| 分类、摘要、提取、问答 | 单个 LLM 调用 | **Claude API** | 一个请求，一个响应 |
| 批处理或嵌入 | 单个 LLM 调用 | **Claude API** | 专用端点 |
| 带代码控制逻辑的多步流水线 | 工作流 | **Claude API + 工具使用** | 您编排循环 |
| 使用自己的工具的自定义 agent | Agent | **Claude API + 工具使用** | 最大灵活性 |
| 带文件/web/终端访问的 AI agent | Agent | **Agent SDK** | 内置工具、安全性和 MCP 支持 |
| Agentic 编码助手 | Agent | **Agent SDK** | 专为该用例设计 |
| 需要内置权限和护栏 | Agent | **Agent SDK** | 包含安全功能 |

> **注意：** Agent SDK 适用于您想要开箱即用的内置文件/web/终端工具、权限和 MCP 的情况。如果您想构建带有自己的工具的 agent，Claude API 是正确的选择 — 使用工具运行器进行自动循环处理，或使用手动循环进行细粒度控制（批准门、自定义日志记录、条件执行）。

### 决策树

```
您的应用程序需要什么？

1. 单个 LLM 调用（分类、摘要、提取、问答）
   └── Claude API — 一个请求，一个响应

2. Claude 是否需要读取/写入文件、浏览 web 或运行 shell 命令
   作为其工作的一部分？（不是：您的应用程序读取文件并将其交给 Claude —
   Claude 本身是否需要发现和访问文件/web/shell？）
   └── 是 → Agent SDK — 内置工具，不要重新实现它们
       示例："扫描代码库查找 bug"、"总结目录中的每个文件"、
             "使用 subagent 查找 bug"、"通过 web 搜索研究主题"

3. 工作流（多步、代码编排、使用自己的工具）
   └── 带工具使用的 Claude API — 您控制循环

4. 开放式 agent（模型决定自己的轨迹、您自己的工具）
   └── Claude API agentic 循环（最大灵活性）
```

### 我应该构建 Agent 吗？

在选择 agent 层之前，检查所有四个标准：

- **复杂性** — 任务是否多步且难以提前完全指定？（例如，"将此设计文档转换为 PR" vs "从此 PDF 中提取标题"）
- **价值** — 结果是否证明更高的成本和延迟是合理的？
- **可行性** — Claude 是否能够胜任此任务类型？
- **错误成本** — 错误是否可以被捕获和恢复？（测试、审查、回滚）

如果其中任何一个是"否"，请保持在更简单的层（单次调用或工作流）。

---

## 架构

所有内容都通过 `POST /v1/messages`。工具和输出约束是这个单一端点的功能 — 而不是单独的 API。

**用户定义工具** — 您定义工具（通过装饰器、Zod schema 或原始 JSON），SDK 的工具运行器处理调用 API、执行您的函数并循环直到 Claude 完成。对于完全控制，您可以手动编写循环。

**服务端工具** — 在 Anthropic 基础设施上运行的 Anthropic 托管工具。代码执行完全在服务端（在 `tools` 中声明，Claude 自动运行代码）。计算机使用可以是 Anthropic 托管的或自托管的。

**结构化输出** — 约束 Messages API 响应格式（`output_config.format`）和/或工具参数验证（`strict: true`）。推荐的方法是 `client.messages.parse()`，它会自动根据您的 schema 验证响应。注意：旧的 `output_format` 参数已弃用；在 `messages.create()` 上使用 `output_config: {format: {...}}`。

**支持端点** — Batches（`POST /v1/messages/batches`）、Files（`POST /v1/files`）和 Token 计数馈入或支持 Messages API 请求。

---

## 当前模型（缓存：2026-02-17）

| 模型 | 模型 ID | 上下文 | 输入 $/1M | 输出 $/1M |
| ----------------- | ------------------- | -------------- | ---------- | ----------- |
| Claude Opus 4.6 | `claude-opus-4-6` | 200K (1M beta) | $5.00 | $25.00 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K (1M beta) | $3.00 | $15.00 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1.00 | $5.00 |

**始终使用 `{{OPUS_ID}}`，除非用户明确指定不同的模型。** 这是不可协商的。不要使用 `{{SONNET_ID}}`、`{{PREV_SONNET_ID}}` 或任何其他模型，除非用户字面上说 "use sonnet" 或 "use haiku"。永远不要为了成本而降级 — 这是用户的决定，而不是您的。

**关键：仅使用上表中精确的模型 ID 字符串 — 它们按原样完整。不要追加日期后缀。** 例如，使用 `claude-sonnet-4-5`，永远不要使用 `claude-sonnet-4-5-20250514` 或您可能从训练数据中回忆的任何其他日期后缀变体。如果用户请求表中未包含的较旧模型（例如，"opus 4.5"、"sonnet 3.7"），请阅读 `shared/models.md` 了解精确 ID — 不要自己构建一个。

注意：如果上面的任何模型字符串看起来对您来说不熟悉，这是意料之中的 — 这只是意味着它们是在您的训练数据截止之后发布的。请放心，它们是真实的模型；我们不会那样戏弄您。

---

## 思考与 Effort（快速参考）

**Opus 4.6 — 自适应思考（推荐）：** 使用 `thinking: {type: "adaptive"}`。Claude 动态决定何时思考以及思考多少。不需要 `budget_tokens` — 在 Opus 4.6 和 Sonnet 4.6 上 `budget_tokens` 已弃用，必须不使用。自适应思考还会自动启用交错思考（无需 beta header）。**当用户要求"扩展思考"、"思考预算"或 `budget_tokens` 时：始终使用 Opus 4.6 和 `thinking: {type: "adaptive"}`。固定 token 预算思考的概念已弃用 — 自适应思考取而代之。不要使用 `budget_tokens`，不要切换到较旧的模型。**

**Effort 参数（GA，无 beta header）：** 通过 `output_config: {effort: "low"|"medium"|"high"|"max"}`（在 `output_config` 内，而非顶级）控制思考深度和整体 token 花费。默认是 `high`（相当于省略它）。`max` 仅适用于 Opus 4.6。适用于 Opus 4.5、Opus 4.6 和 Sonnet 4.6。在 Sonnet 4.5 / Haiku 4.5 上会出错。结合自适应思考以获得最佳成本-质量权衡。对子代理或简单任务使用 `low`；对最深的推理使用 `max`。

**Sonnet 4.6：** 支持自适应思考（`thinking: {type: "adaptive"}`）。在 Sonnet 4.6 上 `budget_tokens` 已弃用 — 使用自适应思考。

**较旧模型（仅当明确请求时）：** 如果用户特别要求 Sonnet 4.5 或其他较旧模型，使用 `thinking: {type: "enabled", budget_tokens: N}`。`budget_tokens` 必须小于 `max_tokens`（最小 1024）。永远不要仅仅因为用户提到 `budget_tokens` 就选择较旧的模型 — 而是使用 Opus 4.6 和自适应思考。

---

## 压缩（快速参考）

**Beta、Opus 4.6 和 Sonnet 4.6。** 对于可能超过 200K 上下文窗口的长时间运行的对话，启用服务端压缩。当接近触发阈值（默认：150K tokens）时，API 会自动总结较早的上下文。需要 beta header `compact-2026-01-12`。

**关键：** 在每个轮次中将 `response.content`（而不仅仅是文本）附加回您的消息。响应中的压缩块必须保留 — API 使用它们在下一个请求中替换压缩的历史记录。仅提取文本字符串并附加它会静默丢失压缩状态。

参见 `{lang}/claude-api/README.md`（压缩部分）了解代码示例。通过 `shared/live-sources.md` 中的 WebFetch 获取完整文档。

---

## 阅读指南

检测语言后，根据用户需求阅读相关文件：

### 快速任务参考

**单次文本分类/摘要/提取/问答：**
→ 仅阅读 `{lang}/claude-api/README.md`

**聊天 UI 或实时响应显示：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**长时间运行的对话（可能超过上下文窗口）：**
→ 阅读 `{lang}/claude-api/README.md` — 参见压缩部分

**函数调用 / 工具使用 / agents：**
→ 阅读 `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**批处理（对延迟不敏感）：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**跨多个请求的文件上传：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**带内置工具的 Agent（文件/web/终端）：**
→ 阅读 `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

### Claude API（完整文件参考）

阅读**特定语言的 Claude API 文件夹**（`{language}/claude-api/`）：

1. **`{language}/claude-api/README.md`** — **先阅读这个。** 安装、快速入门、常见模式、错误处理。
2. **`shared/tool-use-concepts.md`** — 当用户需要函数调用、代码执行、内存或结构化输出时阅读。涵盖概念基础。
3. **`{language}/claude-api/tool-use.md`** — 阅读特定语言的工具使用代码示例（工具运行器、手动循环、代码执行、内存、结构化输出）。
4. **`{language}/claude-api/streaming.md`** — 在构建聊天 UI 或增量显示响应的界面时阅读。
5. **`{language}/claude-api/batches.md`** — 在离线处理许多请求（对延迟不敏感）时阅读。以 50% 成本异步运行。
6. **`{language}/claude-api/files-api.md`** — 在跨多个请求发送同一文件而无需重新上传时阅读。
7. **`shared/error-codes.md`** — 在调试 HTTP 错误或实现错误处理时阅读。
8. **`shared/live-sources.md`** — 用于获取最新官方文档的 WebFetch URL。

> **注意：** 对于 Java、Go、Ruby、C#、PHP 和 cURL — 这些都有单个文件涵盖所有基础知识。阅读该文件以及根据需要阅读 `shared/tool-use-concepts.md` 和 `shared/error-codes.md`。

### Agent SDK

阅读**特定语言的 Agent SDK 文件夹**（`{language}/agent-sdk/`）。Agent SDK 仅适用于 **Python 和 TypeScript**。

1. **`{language}/agent-sdk/README.md`** — 安装、快速入门、内置工具、权限、MCP、hooks。
2. **`{language}/agent-sdk/patterns.md`** — 自定义工具、hooks、子代理、MCP 集成、会话恢复。
3. **`shared/live-sources.md`** — 当前 Agent SDK 文档的 WebFetch URL。

---

## 何时使用 WebFetch

在以下情况下使用 WebFetch 获取最新文档：

- 用户要求"最新"或"当前"信息
- 缓存的数据似乎不正确
- 用户询问此处未涵盖的功能

实时文档 URL 在 `shared/live-sources.md` 中。

## 常见陷阱

- 在将文件或内容传递给 API 时不要截断输入。如果内容太长而无法放入上下文窗口，请通知用户并讨论选项（分块、摘要等），而不是静默截断。
- **Opus 4.6 / Sonnet 4.6 思考：** 使用 `thinking: {type: "adaptive"}` — 不要使用 `budget_tokens`（在 Opus 4.6 和 Sonnet 4.6 上已弃用）。对于较旧模型，`budget_tokens` 必须小于 `max_tokens`（最小 1024）。如果弄错了会抛出错误。
- **Opus 4.6 预填充已移除：** 助手消息预填充（最后助手轮次预填充）在 Opus 4.6 上返回 400 错误。请改用结构化输出（`output_config.format`）或系统提示指令来控制响应格式。
- **128K 输出 tokens：** Opus 4.6 支持多达 128K `max_tokens`，但 SDK 需要对大 `max_tokens` 进行流式传输以避免 HTTP 超时。将 `.stream()` 与 `.get_final_message()` / `.finalMessage()` 一起使用。
- **工具调用 JSON 解析（Opus 4.6）：** Opus 4.6 可能会在工具调用 `input` 字段中产生不同的 JSON 字符串转义（例如，Unicode 或斜杠转义）。始终使用 `json.loads()` / `JSON.parse()` 解析工具输入 — 永远不要对序列化输入进行原始字符串匹配。
- **结构化输出（所有模型）：** 在 `messages.create()` 上使用 `output_config: {format: {...}}` 而不是已弃用的 `output_format` 参数。这是一般的 API 更改，而非 4.6 特定。
- **不要重新实现 SDK 功能：** SDK 提供高级辅助器 — 使用它们而不是从头开始构建。具体来说：使用 `stream.finalMessage()` 而不是将 `.on()` 事件包装在 `new Promise()` 中；使用类型化异常类（`Anthropic.RateLimitError` 等）而不是字符串匹配错误消息；使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.Message` 等）而不是重新定义等效接口。
- **不要为 SDK 数据结构定义自定义类型：** SDK 导出所有 API 对象的类型。对消息使用 `Anthropic.MessageParam`，对工具定义使用 `Anthropic.Tool`，对工具结果使用 `Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam`，对响应使用 `Anthropic.Message`。定义自己的 `interface ChatMessage { role: string; content: unknown }` 重复了 SDK 已经提供的内容，并失去了类型安全性。
- **报告和文档输出：** 对于生成报告、文档或可视化的任务，代码执行沙箱预装了 `python-docx`、`python-pptx`、`matplotlib`、`pillow` 和 `pypdf`。Claude 可以生成格式化文件（DOCX、PDF、图表）并通过 Files API 返回它们 — 对于"报告"或"文档"类型请求请考虑这一点，而不是纯 stdout 文本。
