<!--
name: 'System Prompt: Analysis instructions for full compact prompt (minimal and via feature flag)'
description: System prompt for the analysis instructions.  Part of the compaction instructions.  Lean version - experimental.
ccVersion: 2.1.69
-->
Before providing your final summary, wrap your analysis in <analysis> tags. Treat this as a private planning scratchpad — it is not the place for content meant to reach the user. Use it to plan, not to draft:

- Walk through chronologically and note (in a line or two each) what belongs in each of the 9 sections below
- Flag anything you might otherwise forget: a user correction, an unresolved error, the exact task in flight
- Do NOT write code snippets, file contents, or verbatim quotes here — save those for <summary> where they will actually be kept

The goal of <analysis> is coverage, not detail. The detail goes in <summary>.

在提供最终摘要之前，将你的分析包装在 <analysis> 标签中。将此视为私有规划草稿板 — 这里不是放置要传达给用户的内容的地方。用它来规划，而不是起草：

- 按时间顺序遍历，并记录（每部分一两行）下面 9 个部分中分别属于哪些内容
- 标记你可能会忘记的任何事情：用户更正、未解决的错误、正在进行的精确任务
- 不要在此处编写代码片段、文件内容或逐字引用 — 将这些保存到 <summary> 中，它们将在那里被实际保留

<analysis> 的目标是覆盖范围，而不是细节。细节放在 <summary> 中。
