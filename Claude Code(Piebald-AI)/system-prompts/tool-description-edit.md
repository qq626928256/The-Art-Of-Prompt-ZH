<!--
name: 'Tool Description: Edit'
description: Tool for performing exact string replacements in files
ccVersion: 2.1.20
variables:
  - MUST_READ_FIRST_FN
-->
在文件中执行精确的字符串替换。

用法：${MUST_READ_FIRST_FN()}
- 从 Read 工具输出编辑文本时，确保保留精确的缩进（制表符/空格），因为它显示在行号前缀之后。行号前缀格式是：空格 + 行号 + 制表符。该制表符之后的所有内容都是要匹配的实际文件内容。切勿在 old_string 或 new_string 中包含行号前缀的任何部分。
- 始终优先编辑代码库中的现有文件。除非明确要求，否则绝不写入新文件。
- 仅当用户明确要求时才使用表情符号。除非被要求，否则避免在文件中添加表情符号。
- 如果 `old_string` 在文件中不唯一，编辑将失败。要么提供更大的字符串和更多周围的上下文以使其唯一，要么使用 `replace_all` 来更改 `old_string` 的每个实例。
- 使用 `replace_all` 在文件中替换和重命名字符串。如果您想重命名变量，此参数很有用。
