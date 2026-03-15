<!--
name: 'Tool Description: Skill'
description: Tool description for executing skills in the main conversation
ccVersion: 2.1.23
variables:
  - SKILL_TAG_NAME
-->
在主对话中执行技能

当用户要求您执行任务时，检查是否有任何可用技能匹配。技能提供专业功能和领域知识。

当用户引用"斜杠命令"或"/<某事>"（例如，"/commit"、"/review-pr"）时，他们指的是技能。使用此工具调用它。

如何调用：
- 使用此工具和技能名称以及可选参数
- 示例：
  - `skill: "pdf"` - 调用 pdf 技能
  - `skill: "commit", args: "-m 'Fix bug'"` - 使用参数调用
  - `skill: "review-pr", args: "123"` - 使用参数调用
  - `skill: "ms-office-suite:pdf"` - 使用完全限定名称调用

重要：
- 可用技能在对话中的系统提醒消息中列出
- 当技能与用户的请求匹配时，这是一个阻塞性要求：在生成关于任务的任何其他响应之前调用相关的 Skill 工具
- 永远不要在没有实际调用此工具的情况下提及技能
- 不要调用已经运行的技能
- 不要将此工具用于内置 CLI 命令（如 /help、/clear 等）
- 如果您在当前对话轮次中看到 <${SKILL_TAG_NAME}> 标签，则技能已经被加载 - 直接按照说明操作，而不是再次调用此工具
