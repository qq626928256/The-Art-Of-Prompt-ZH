<!--
name: 'Data: GitHub App installation PR description'
description: Template for PR description when installing Claude Code GitHub App integration
ccVersion: 2.0.14
-->
## 🤖 安装 Claude Code GitHub App

此 PR 添加了一个 GitHub Actions 工作流程，使我们仓库能够集成 Claude Code。

### 什么是 Claude Code？

[Claude Code](https://claude.com/claude-code) 是一个 AI 编码 Agent，可以帮助：
- Bug 修复和改进
- 文档更新
- 实现新功能
- 代码审查和建议
- 编写测试
- 以及更多！

### 工作原理

合并此 PR 后，我们将能够在拉取请求或问题评论中提及 @claude 来与 Claude 交互。
触发工作流程后，Claude 将分析评论和周围上下文，并在 GitHub 操作中执行请求。

### 重要说明

- **此工作流程在合并此 PR 之前不会生效**
- **@claude 提及在合并完成后才能工作**
- 每当在 PR 或问题评论中提及 Claude 时，工作流程会自动运行
- Claude 可以访问整个 PR 或问题上下文，包括文件、差异和之前的评论

### 安全性

- 我们的 Anthropic API 密钥安全地存储为 GitHub Actions 密钥
- 只有对仓库具有写入权限的用户才能触发工作流程
- 所有 Claude 运行都存储在 GitHub Actions 运行历史中
- Claude 的默认工具仅限于读取/写入文件，以及通过创建评论、分支和提交与我们的仓库交互。
- 我们可以通过在workflow文件中添加更多允许的工具：

\`\`\`
allowed_tools: Bash(npm install),Bash(npm run build),Bash(npm run lint),Bash(npm run test)
\`\`\`

更多信息请参见 [Claude Code action 仓库](https://github.com/anthropics/claude-code-action)。

合并此 PR 后，让我们在任何 PR 的评论中提及 @claude 来开始使用！
