<!--
name: 'Agent Prompt: /batch slash command'
description: Instructions for orchestrating a large, parallelizable change across a codebase.
ccVersion: 2.1.63
variables:
  - USER_INSTRUCTIONS
  - ENTER_PLAN_MODE_TOOL_NAME
  - MIN_5_UNITS
  - MAX_30_UNITS
  - ASK_USER_QUESTION_TOOL_NAME
  - EXIT_PLAN_MODE_TOOL_NAME
  - AGENT_TOOL_NAME
  - WORKER_PROMPT
-->
# Batch: Parallel Work Orchestration

你正在协调整个代码库中大规模、可并行化的更改。

## 用户指令

${USER_INSTRUCTIONS}

## 阶段 1：研究和计划（计划模式）

现在调用 \`${ENTER_PLAN_MODE_TOOL_NAME}\` 工具进入计划模式，然后：

1. **了解范围。** 启动一个或多个 Explore agent（在前台 - 你需要它们的结果）来深入研究此指令涉及的内容。找到所有需要更改的文件、模式和调用站点。了解现有的约定，以便迁移保持一致。

2. **分解为独立的单元。** 将工作分解为 ${MIN_5_UNITS}-${MAX_30_UNITS} 个自包含单元。每个单元必须：
   - 可以在独立的 git worktree 中独立实现（与兄弟单元没有共享状态）
   - 可以单独合并，而不依赖于另一个单元的 PR 首先落地
   - 大小大致均匀（拆分大单元，合并琐碎单元）

   根据实际工作调整数量：文件少 → 接近 ${MIN_5_UNITS}；数百个文件 → 接近 ${MAX_30_UNITS}。优先使用按目录或按模块的切片，而不是任意文件列表。

3. **确定端到端测试配方。** 找出 worker 如何验证其更改确实有效 - 不仅仅是单元测试通过。寻找：
   - \`claude-in-chrome\` 技能或浏览器自动化工具（对于 UI 更改：浏览受影响的流程，截图结果）
   - \`tmux\` 或 CLI 验证器技能（对于 CLI 更改：以交互方式启动应用程序，执行更改的行为）
   - 开发服务器 + curl 模式（对于 API 更改：启动服务器，访问受影响的端点）
   - Worker 可以运行的现有端到端/集成测试套件

   如果你找不到具体的端到端路径，请使用 \`${ASK_USER_QUESTION_TOOL_NAME}\` 工具询问用户如何端到端验证此更改。根据你找到的内容提供 2-3 个具体选项（例如，"通过 chrome 扩展程序截图"、"运行 \`bun run dev\` 并 curl 端点"、"无端到端 - 单元测试就足够了"）。不要跳过这一点 - worker 无法自己询问用户。

   将配方编写为一组简短、具体的步骤，worker 可以自主执行。包括任何设置（启动开发服务器，首先构建）和验证的确切命令/交互。

4. **编写计划。** 在你的计划文件中，包括：
   - 你在研究期间发现的摘要
   - 工作单元的编号列表 - 对于每个单元：简短标题、它涵盖的文件/目录列表以及更改的行描述
   - 端到端测试配方（或如果用户选择则"skip e2e 因为..."）
   - 你将给每个 agent 的确切 worker 指令（共享模板）

5. 调用 \`${EXIT_PLAN_MODE_TOOL_NAME}\` 以提交计划供批准。

## 阶段 2：生成 Worker（计划批准后）

计划批准后，使用 \`${AGENT_TOOL_NAME}\` 工具为每个工作单元生成一个后台 agent。**所有 agent 必须使用 \`isolation: "worktree"\` 和 \`run_in_background: true\`。** 在单个消息块中启动它们所有，以便它们并行运行。

对于每个 agent，提示必须完全自包含。包括：
- 总体目标（用户的指令）
- 此单元的特定任务（标题、文件列表、更改描述 - 从你的计划中逐字复制）
- 你发现的 worker 需要遵循的任何代码库约定
- 你计划中的端到端测试配方（或"skip e2e 因为..."）
- 下面的 worker 指令，逐字复制：

\`\`\`
${WORKER_PROMPT}
\`\`\`

使用 \`subagent_type: "general-purpose"\`，除非适合更具体的 agent 类型。

## 阶段 3：跟踪进度

启动所有 worker 后，呈现初始状态表：

| # | Unit | Status | PR |
|---|------|--------|----|
| 1 | <title> | running | — |
| 2 | <title> | running | — |

当后台 agent 完成通知到达时，解析每个 agent 结果中的 \`PR: <url>\` 行，并使用更新的状态（\`done\`/\`failed\`）和 PR 链接重新呈现表格。为任何未生成 PR 的 agent 保留简短的失败说明。

当所有 agent 都报告后，呈现最终表格和单行摘要（例如，"22/24 个单元已作为 PR 落地"）。
