<!--
name: 'Agent Prompt: Recent Message Summarization'
description: Agent prompt used for summarizing recent messages.
ccVersion: 2.1.69
variables:
  - ANALYSIS_INSTRUCTION_TAGS
-->
你的任务是创建对话最近部分的详细摘要 - 跟随较早保留上下文的消息。较早的消息保持完整，不需要摘要。将你的摘要仅集中在最近消息中讨论、学习和完成的内容上。

${ANALYSIS_INSTRUCTION_TAGS}

你的摘要应包括以下部分：

1. 主要请求和意图：捕获来自最近消息的用户的明确请求和意图
2. 关键技术概念：列出最近讨论的重要技术概念、技术和框架。
3. 文件和代码部分：枚举检查、修改或创建的特定文件和代码部分。在适用处包括完整的代码片段，并包括关于为什么此文件读取或编辑很重要的摘要。
4. 错误和修复：列出遇到的错误以及它们是如何修复的。
5. 问题解决：记录已解决的问题和任何正在进行的故障排除工作。
6. 所有用户消息：列出最近部分中所有非工具结果的用户消息。
7. 待处理任务：概述最近消息中的任何待处理任务。
8. 当前工作：精确描述在此摘要请求之前正在进行的工作。
9. 可选的下一步：列出与最近的工作相关的下一步。包括最近对话中的直接引用。

以下是你的输出应如何结构的示例：

<example>
<analysis>
[你的思维过程，确保所有点都被彻底和准确地涵盖]
</analysis>

<summary>
1. Primary Request and Intent:
   [详细描述]

2. Key Technical Concepts:
   - [概念 1]
   - [概念 2]

3. Files and Code Sections:
   - [文件名 1]
      - [关于此文件为何重要的摘要]
      - [重要代码片段]

4. Errors and fixes:
    - [错误描述]:
      - [你如何修复它]

5. Problem Solving:
   [描述]

6. All user messages:
    - [详细的非工具使用用户消息]

7. Pending Tasks:
   - [任务 1]

8. Current Work:
   [当前工作的精确描述]

9. Optional Next Step:
   [要采取的可选下一步]

</summary>
</example>

请仅基于最近的消息（在保留的较早上下文之后）提供你的摘要，遵循此结构并确保响应的精确性和彻底性。
