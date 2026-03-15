<!--
name: 'Data: Live documentation sources'
description: WebFetch URLs for fetching current Claude API and Agent SDK documentation from official sources
ccVersion: 2.1.63
-->
# 实时文档源

此文件包含用于从 platform.claude.com 和 Agent SDK 仓库获取最新信息的 WebFetch URL。当用户需要自缓存内容上次更新以来可能已更改的最新数据时，请使用这些 URL。

## 何时使用 WebFetch

- 用户明确要求"最新"或"当前"信息
- 缓存数据似乎不正确
- 用户询问缓存内容中未涵盖的功能
- 用户需要特定的 API 详细信息或示例

## Claude API 文档 URL

### 模型和定价

| 主题           | URL                                                                   | 提取提示                                                               |
| --------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 模型概述 | \`https://platform.claude.com/docs/en/about-claude/models/overview.md\` | "Extract current model IDs, context windows, and pricing for all Claude models" |
| 定价         | \`https://platform.claude.com/docs/en/pricing.md\`                      | "Extract current pricing per million tokens for input and output"               |

### 核心功能

| 主题             | URL                                                                          | 提示提示                                                                      |
| ----------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| 扩展思考 | \`https://platform.claude.com/docs/en/build-with-claude/extended-thinking.md\` | "Extract extended thinking parameters, budget_tokens requirements, and usage examples" |
| 自适应思考 | \`https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking.md\` | "Extract adaptive thinking setup, effort levels, and {{OPUS_NAME}} usage examples"         |
| Effort 参数  | \`https://platform.claude.com/docs/en/build-with-claude/effort.md\`            | "Extract effort levels, cost-quality tradeoffs, and interaction with thinking"        |
| 工具使用          | \`https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview.md\`  | "Extract tool definition schema, tool_choice options, and handling tool results"       |
| 流式传输         | \`https://platform.claude.com/docs/en/build-with-claude/streaming.md\`         | "Extract streaming event types, SDK examples, and best practices"                      |
| 提示缓存    | \`https://platform.claude.com/docs/en/build-with-claude/prompt-caching.md\`    | "Extract cache_control usage, pricing benefits, and implementation examples"           |

### 媒体和文件

| 主题       | URL                                                                    | 提取提示                                                 |
| ----------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 视觉      | \`https://platform.claude.com/docs/en/build-with-claude/vision.md\`      | "Extract supported image formats, size limits, and code examples" |
| PDF 支持 | \`https://platform.claude.com/docs/en/build-with-claude/pdf-support.md\` | "Extract PDF handling capabilities, limits, and examples"         |

### API 操作

| 主题            | URL                                                                         | 提取提示                                                                                       |
| ---------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| 批处理 | \`https://platform.claude.com/docs/en/build-with-claude/batch-processing.md\` | "Extract batch API endpoints, request format, and polling for results"                                  |
| Files API        | \`https://platform.claude.com/docs/en/build-with-claude/files.md\`            | "Extract file upload, download, and referencing in messages, including supported types and beta header" |
| Token 计数   | \`https://platform.claude.com/docs/en/build-with-claude/token-counting.md\`   | "Extract token counting API usage and examples"                                                         |
| 速率限制      | \`https://platform.claude.com/docs/en/api/rate-limits.md\`                    | "Extract current rate limits by tier and model"                                                         |
| 错误           | \`https://platform.claude.com/docs/en/api/errors.md\`                         | "Extract HTTP error codes, meanings, and retry guidance"                                                |

### 工具

| 主题          | URL                                                                                    | 提取提示                                                                        |
| -------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 代码执行 | \`https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool.md\` | "Extract code execution tool setup, file upload, container reuse, and response handling" |
| 计算机使用   | \`https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use.md\`        | "Extract computer use tool setup, capabilities, and implementation examples"             |

### 高级功能

| 主题              | URL                                                                           | 提取提示                                   |
| ------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------- |
| 结构化输出 | \`https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md\` | "Extract output_config.format usage and schema enforcement"                           |
| 压缩         | \`https://platform.claude.com/docs/en/build-with-claude/compaction.md\`         | "Extract compaction setup, trigger config, and streaming with compaction"             |
| 引用          | \`https://platform.claude.com/docs/en/build-with-claude/citations.md\`          | "Extract citation format and implementation"        |
| 上下文窗口    | \`https://platform.claude.com/docs/en/build-with-claude/context-windows.md\`    | "Extract context window sizes and token management" |

---

## Claude API SDK 仓库

| SDK        | URL                                                       | 描述                    |
| ---------- | --------------------------------------------------------- | ------------------------------ |
| Python     | \`https://github.com/anthropics/anthropic-sdk-python\`     | \`anthropic\` pip package source |
| TypeScript | \`https://github.com/anthropics/anthropic-sdk-typescript\` | \`@anthropic-ai/sdk\` npm source |
| Java       | \`https://github.com/anthropics/anthropic-sdk-java\`       | \`anthropic-java\` Maven source  |
| Go         | \`https://github.com/anthropics/anthropic-sdk-go\`         | Go module source               |
| Ruby       | \`https://github.com/anthropics/anthropic-sdk-ruby\`       | \`anthropic\` gem source         |
| C#         | \`https://github.com/anthropics/anthropic-sdk-csharp\`     | NuGet package source           |
| PHP        | \`https://github.com/anthropics/anthropic-sdk-php\`        | Composer package source        |

---

## Agent SDK 文档 URL

### 核心文档

| 主题                | URL                                                         | 提取提示                                               |
| -------------------- | ----------------------------------------------------------- | --------------------------------------------------------------- |
| Agent SDK 概览   | \`https://platform.claude.com/docs/en/agent-sdk.md\`          | "Extract the Agent SDK overview, key features, and use cases"   |
| Agent SDK Python     | \`https://github.com/anthropics/claude-agent-sdk-python\`     | "Extract Python SDK installation, imports, and basic usage"     |
| Agent SDK TypeScript | \`https://github.com/anthropics/claude-agent-sdk-typescript\` | "Extract TypeScript SDK installation, imports, and basic usage" |

### SDK 参考 (GitHub README)

| 主题          | URL                                                                                       | 提取提示                                            |
| -------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Python SDK     | \`https://raw.githubusercontent.com/anthropics/claude-agent-sdk-python/main/README.md\`     | "Extract Python SDK API reference, classes, and methods"     |
