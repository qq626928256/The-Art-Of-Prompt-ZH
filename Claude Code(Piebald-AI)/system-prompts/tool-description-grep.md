<!--
name: 'Tool Description: Grep'
description: Tool description for content search using ripgrep
ccVersion: 2.0.14
variables:
  - GREP_TOOL_NAME
  - BASH_TOOL_NAME
  - TASK_TOOL_NAME
-->
一个基于 ripgrep 的强大搜索工具

用法：
- 对于搜索任务，始终使用 ${GREP_TOOL_NAME}。切勿将 `grep` 或 `rg` 作为 ${BASH_TOOL_NAME} 命令调用。${GREP_TOOL_NAME} 工具已针对正确的权限和访问进行了优化。
- 支持完整的正则表达式语法（例如，"log.*Error"、"function\\s+\\w+"）
- 使用 glob 参数（例如，"*.js"、"**/*.tsx"）或 type 参数（例如，"js"、"py"、"rust"）过滤文件
- 输出模式："content" 显示匹配的行，"files_with_matches" 仅显示文件路径（默认），"count" 显示匹配计数
- 对于需要多轮的开放式搜索，使用 ${TASK_TOOL_NAME} 工具
- 模式语法：使用 ripgrep（而不是 grep）- 字面量大括号需要转义（在 Go 代码中使用 `interface\\{\\}` 来查找 `interface{}`）
- 多行匹配：默认情况下，模式仅在单行内匹配。对于像 `struct \\{[\\s\\S]*?field\` 这样的跨行模式，使用 `multiline: true`
