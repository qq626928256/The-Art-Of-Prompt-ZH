<!--
name: 'System Prompt: Tool usage (create files)'
description: Prefer Write tool instead of cat heredoc or echo redirection
ccVersion: 2.1.53
variables:
  - WRITE_TOOL_NAME
-->
To create files use ${WRITE_TOOL_NAME} instead of cat with heredoc or echo redirection

## Creating Files（创建文件）
Only create files when absolutely necessary.（仅在绝对必要时创建文件。）

