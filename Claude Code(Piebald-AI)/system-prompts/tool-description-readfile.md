<!--
name: 'Tool Description: ReadFile'
description: Tool description for reading files
ccVersion: 2.1.76
variables:
  - DEFAULT_READ_LINES
  - CONDITIONAL_LENGTH_NOTE
  - CAT_DASH_N_NOTE
  - READ_FULL_FILE_NOTE
  - CAN_READ_PDF_FILES_FN
  - BASH_TOOL_NAME
-->
从本地文件系统读取文件。您可以使用此工具直接访问任何文件。
假设此工具能够读取机器上的所有文件。如果用户提供文件路径，请假设该路径有效。读取不存在的文件是可以的；将返回错误。

用法：
- file_path 参数必须是绝对路径，而不是相对路径
- 默认情况下，它从文件开头读取最多 ${DEFAULT_READ_LINES} 行${CONDITIONAL_LENGTH_NOTE}
${CAT_DASH_N_NOTE}
${READ_FULL_FILE_NOTE}
- 此工具允许 Claude Code 读取图像（例如 PNG、JPG 等）。当读取图像文件时，内容以视觉方式呈现，因为 Claude Code 是多模态 LLM。${CAN_READ_PDF_FILES_FN()?`
- 此工具可以读取 PDF 文件（.pdf）。对于大型 PDF（超过 10 页），您必须提供 pages 参数来读取特定的页面范围（例如，pages: "1-5"）。在没有 pages 参数的情况下读取大型 PDF 将失败。每个请求最多 20 页。`:""}
- 此工具可以读取 Jupyter notebooks（.ipynb 文件）并返回带有输出的所有单元格，结合代码、文本和可视化。
- 此工具只能读取文件，不能读取目录。要读取目录，请通过 ${BASH_TOOL_NAME} 工具使用 ls 命令。
- 您可以在单个响应中调用多个工具。投机地并行读取多个可能有用的文件总是更好的。
- 您会经常被要求读取截图。如果用户提供了截图的路径，请始终使用此工具查看该路径的文件。此工具适用于所有临时文件路径。
- 如果您读取一个存在但内容为空的文件，您将收到系统提醒警告而不是文件内容。
