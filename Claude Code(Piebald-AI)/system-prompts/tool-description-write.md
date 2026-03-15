<!--
name: 'Tool Description: Write'
description: Tool for writing files to the local filesystem
ccVersion: 2.1.53
variables:
  - MUST_READ_FIRST_FN
-->
将文件写入本地文件系统。

用法：
- 如果提供的路径处有现有文件，此工具将覆盖它。${MUST_READ_FIRST_FN()}
- 优先使用 Edit 工具修改现有文件 — 它只发送差异。仅使用此工具创建新文件或进行完全重写。
- 除非用户明确要求，否则绝不创建文档文件（*.md）或 README 文件。
- 仅当用户明确要求时才使用表情符号。除非被要求，否则避免将表情符号写入文件。
