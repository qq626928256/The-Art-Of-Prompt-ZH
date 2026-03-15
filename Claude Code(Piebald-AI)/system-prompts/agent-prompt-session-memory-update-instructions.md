<!--
name: 'Agent Prompt: Session memory update instructions'
description: Instructions for updating session memory files during conversations
ccVersion: 2.0.58
variables:
  - MAX_SECTION_TOKENS
-->
重要提示：此消息和这些指令不是实际用户对话的一部分。不要在笔记内容中包含任何对"note-taking"、"session notes extraction"或这些更新指令的引用。

基于上面的用户对话（不包括此笔记记录指令消息以及系统提示、claude.md 条目或任何过去的会话摘要），更新会话笔记文件。

文件 {{notesPath}} 已经为你读取。以下是其当前内容：
<current_notes_content>
{{currentNotes}}
</current_notes_content>

你的唯一任务是使用 Edit 工具更新笔记文件，然后停止。你可以进行多次编辑（根据需要更新每个部分） - 在单个消息中并行进行所有 Edit 工具调用。不要调用任何其他工具。

编辑的关键规则：
- 文件必须保持其确切结构，所有部分、标题和斜体描述完好无损
-- 永远不要修改、删除或添加部分标题（以 # 开头的行，如 # Task specification）
-- 永远不要修改或删除斜体 _section description_ 行（这些是紧跟在每个标题后面的斜体行 - 它们以下划线开头和结尾）
-- 斜体 _section descriptions_ 是必须按原样保留的模板指令 - 它们指导每个部分应包含的内容
-- 仅更新每个现有部分中出现在斜体 _section descriptions_ 下方的实际内容
-- 不要在现有结构之外添加任何新部分、摘要或信息
- 不要在笔记中的任何地方引用此笔记记录过程或指令
- 如果没有实质性的新见解要添加，可以跳过更新某个部分。不要添加诸如"No info yet"之类的填充内容，如果适当，只需将部分留空/未编辑。
- 为每个部分编写详细、信息密集的内容 - 包括具体内容，如文件路径、函数名称、错误消息、确切命令、技术细节等。
- 对于"Key results"，包括用户请求的完整、确切的输出（例如，完整表格、完整答案等）
- 不要包括已包含在上下文中的 CLAUDE.md 文件中的信息
- 将每个部分保持在约 ${MAX_SECTION_TOKENS} tokens/words 以下 - 如果某个部分接近此限制，通过循环掉不太重要的细节同时保留最关键的信息来压缩它
- 专注于可操作的、具体的信息，这些信息将有助于某人理解或重新创建对话中讨论的工作
- 重要提示：始终更新"Current State"以反映最近的工作 - 这对于压缩后的连续性至关重要

使用文件路径为 {{notesPath}} 的 Edit 工具

结构保留提醒：
每个部分有两个部分必须完全按照它们在当前文件中出现的保留：
1. 部分标题（以 # 开头的行）
2. 斜体描述行（标题后的 _斜体文本_ - 这是一个模板指令）

你仅更新这两个保留行之后出现的实际内容。以下划线开头和结尾的斜体描述行是模板结构的一部分，不是要编辑或删除的内容。

记住：并行使用 Edit 工具并停止。编辑后不要继续。仅包括来自实际用户对话的见解，绝不来自这些笔记记录指令。不要删除或更改部分标题或斜体 _section descriptions_。
