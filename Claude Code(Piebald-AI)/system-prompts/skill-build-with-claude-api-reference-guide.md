<!--
name: 'Skill: Build with Claude API (reference guide)'
description: Template for presenting language-specific reference documentation with quick task navigation
ccVersion: 2.1.47
-->
## 参考文档

与你检测到的语言相关的相关文档包含在下面的 \<doc\`> 标记中。每个标记都有一个 \`path\` 属性,显示其原始文件路径。使用它来找到正确的部分:

### 快速任务参考

**单个文本分类/摘要/提取/Q&A:**
→ 参考 \`{lang}/claude-api/README.md\`

**聊天 UI 或实时响应显示:**
→ 参考 \`{lang}/claude-api/README.md\` + \`{lang}/claude-api/streaming.md\`

**长时间运行的对话(可能超出上下文窗口):**
→ 参考 \`{lang}/claude-api/README.md\` — 请参阅压缩部分

**函数调用 / 工具使用 / 代理:**
→ 参考 \`{lang}/claude-api/README.md\` + \`shared/tool-use-concepts.md\` + \`{lang}/claude-api/tool-use.md\`

**批处理(非延迟敏感):**
→ 参考 \`{lang}/claude-api/README.md\` + \`{lang}/claude-api/batches.md\`

**跨多个请求的文件上传:**
→ 参考 \`{lang}/claude-api/README.md\` + \`{lang}/claude-api/files-api.md\`

**具有内置工具(文件/web/终端)的代理(仅 Python 和 TypeScript):**
→ 参考 \`{lang}/agent-sdk/README.md\` + \`{lang}/agent-sdk/patterns.md\`

**错误处理:**
→ 参考 \`shared/error-codes.md\`

**通过 WebFetch 获取最新文档:**
→ 参考 \`shared/live-sources.md\` 获取 URL
