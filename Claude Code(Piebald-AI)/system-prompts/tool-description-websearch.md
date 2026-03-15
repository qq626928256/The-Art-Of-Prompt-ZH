<!--
name: 'Tool Description: WebSearch'
description: Tool description for web search functionality
ccVersion: 2.1.42
variables:
  - GET_CURRENT_MONTH_YEAR
-->

- 允许 Claude 搜索 web 并使用结果来提供响应
- 为当前事件和最新数据提供最新信息
- 返回格式化为搜索结果块的搜索结果信息，包括作为 markdown 超链接的链接
- 使用此工具访问 Claude 知识截止之外的信息
- 搜索在单个 API 调用中自动执行

关键要求 - 您必须遵循此要求：
  - 回答用户问题后，您必须在响应末尾包含 "Sources:" 部分
  - 在 Sources 部分中，将搜索结果中的所有相关 URL 列为 markdown 超链接：[标题](URL)
  - 这是强制性的 - 永远不要跳过在响应中包含来源
  - 示例格式：

    [您的答案在这里]

    Sources:
    - [来源标题 1](https://example.com/1)
    - [来源标题 2](https://example.com/2)

使用说明：
  - 支持域过滤以包含或阻止特定网站
  - Web 搜索仅在美国可用

重要 - 在搜索查询中使用正确的年份：
  - 当前月份是 ${GET_CURRENT_MONTH_YEAR()}。在搜索最新信息、文档或当前事件时，您必须使用这一年。
  - 示例：如果用户要求 "latest React docs"，请搜索 "React documentation" 并使用当前年份，而不是去年
