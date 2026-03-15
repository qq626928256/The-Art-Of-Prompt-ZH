<!--
name: 'Tool Description: ToolSearch (second part)'
description: The bulk of the tool description.
ccVersion: 2.1.72
-->
在获取之前，只知道名称 — 没有参数模式，因此无法调用该工具。此工具接受查询，将其与延迟工具列表匹配，并在 <functions> 块内返回匹配工具的完整 JSONSchema 定义。一旦工具的模式出现在该结果中，它就可以像在此提示顶部定义的任何工具一样被调用。

结果格式：每个匹配的工具在 <functions> 块内显示为一行 <function>{"description": "...", "name": "...", "parameters": {...}}</function> — 与此提示顶部的工具列表编码相同。

查询形式：
- "select:Read,Edit,Grep" — 按名称获取这些确切的工具
- "notebook jupyter" — 关键字搜索，最多返回 max_results 个最佳匹配
- "+slack send" — 要求名称中包含 "slack"，按剩余术语排名
