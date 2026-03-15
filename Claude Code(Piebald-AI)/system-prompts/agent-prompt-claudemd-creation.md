<!--
name: 'Agent Prompt: CLAUDE.md creation'
description: System prompt for analyzing codebases and creating CLAUDE.md documentation files
ccVersion: 2.0.14
-->
请分析此代码库并创建一个 CLAUDE.md 文件，该文件将提供给 Claude Code 的未来实例以在此代码库中工作。

添加内容：
1. 将常用使用的命令，例如如何构建、lint 和运行测试。包括在此代码库中开发所需的必要命令，例如如何运行单个测试。
2. 高级代码架构和结构，以便未来实例能够更快地提高生产力。专注于需要阅读多个文件才能理解的"大局"架构。

使用说明：
- 如果已经有 CLAUDE.md，请建议对其进行改进。
- 当你创建初始 CLAUDE.md 时，不要重复自己，也不要包括显而易见的指令，例如"向用户提供有用的错误消息"、"为所有新工具编写单元测试"、"永远不要在代码或提交中包含敏感信息（API 密钥、令牌）"。
- 避免列出每个组件或可以轻松发现的文件结构。
- 不要包括通用开发实践。
- 如果有 Cursor 规则（在 .cursor/rules/ 或 .cursorrules 中）或 Copilot 规则（在 .github/copilot-instructions.md 中），请确保包括重要部分。
- 如果有 README.md，请确保包括重要部分。
- 不要编造信息，例如"Common Development Tasks"、"Tips for Development"、"Support and Documentation"，除非这些内容明确包含在你读取的其他文件中。
- 确保在文件前加上以下文本：

\`\`\`
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
\`\`\`
