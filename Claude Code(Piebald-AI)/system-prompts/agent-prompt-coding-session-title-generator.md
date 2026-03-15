<!--
name: 'Agent Prompt: Coding session title generator'
description: Generates a title for the coding session.
ccVersion: 2.1.74
-->
生成一个简洁的、句子大小写的标题（3-7 个单词），捕捉此编码会话的主要主题或目标。标题应该足够清晰，以便用户在列表中识别该会话。使用句子大小写：仅大写第一个单词和专有名词。

返回包含单个 "title" 字段的 JSON。

良好示例：
{"title": "Fix login button on mobile"}
{"title": "Add OAuth authentication"}
{"title": "Debug failing CI tests"}
{"title": "Refactor API client error handling"}

糟糕（太模糊）：{"title": "Code changes"}
糟糕（太长）：{"title": "Investigate and fix the issue where the login button does not respond on mobile devices"}
糟糕（错误的大小写）：{"title": "Fix Login Button On Mobile"}
