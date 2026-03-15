<!--
name: 'Skill: /loop slash command'
description: Parses user input into an interval and prompt, converts the interval to a cron expression, and schedules a recurring task
ccVersion: 2.1.71
variables:
  - CRON_CREATE_TOOL_NAME
  - DEFAULT_INTERVAL
  - CRON_CANCEL_TOOL_NAME
  - USER_INPUT
-->
# /loop — 安排循环提示

将下面的输入解析为 \`[interval] <prompt…>\` 并使用 ${CRON_CREATE_TOOL_NAME} 安排它。

## 解析(按优先级顺序)

1. **前导标记**:如果第一个空格分隔的标记匹配 \`^\\d+[smhd]$\`(例如 \`5m\`, \`2h\`),那就是间隔;其余部分是提示。
2. **尾随 "every" 子句**:否则,如果输入以 \`every <N><unit>\` 或 \`every <N> <unit-word>\` 结尾(例如 \`every 20m\`, \`every 5 minutes\`, \`every 2 hours\`),将其提取为间隔并从提示中删除。仅当 "every" 后面跟着时间表达式时才匹配 — \`check every PR\` 没有间隔。
3. **默认**:否则,间隔为 \`${DEFAULT_INTERVAL}\` 整个输入是提示。

如果结果提示为空,显示用法 \`/loop [interval] <prompt>\` 并停止 — 不要调用 ${CRON_CREATE_TOOL_NAME}。

示例:
- \`5m /babysit-prs\` → 间隔 \`5m\`,提示 \`/babysit-prs\`(规则 1)
- \`check the deploy every 20m\` → 间隔 \`20m\`,提示 \`check the deploy\`(规则 2)
- \`run tests every 5 minutes\` → 间隔 \`5m\`,提示 \`run tests\`(规则 2)
- \`check the deploy\` → 间隔 \`${DEFAULT_INTERVAL}\`,提示 \`check the deploy\`(规则 3)
- \`check every PR\` → 间隔 \`${DEFAULT_INTERVAL}\`,提示 \`check every PR\`(规则 3 — "every" 后面没有时间)
- \`5m\` → 空提示 → 显示用法

## 间隔 → cron

支持的后缀:\`s\`(秒,向上舍入到最接近的分钟,最小 1),\`m\`(分钟),\`h\`(小时),\`d\`(天)。转换:

| 间隔模式         | Cron 表达式          | 说明                                          |
| ---------------- | -------------------- | --------------------------------------------- |
| \`Nm\` 其中 N ≤ 59   | \`*/N * * * *\`       | 每 N 分钟                                     |
| \`Nm\` 其中 N ≥ 60   | \`0 */H * * *\`       | 舍入到小时(H = N/60,必须能整除 24)           |
| \`Nh\` 其中 N ≤ 23   | \`0 */N * * *\`       | 每 N 小时                                     |
| \`Nd\`                | \`0 0 */N * *\`       | 每 N 天在午夜本地时间                         |
| \`Ns\`                | 视为 \`ceil(N/60)m\`   | cron 最小粒度为 1 分钟                        |

**如果间隔不能干净地整除其单位**(例如 \`7m\` → \`*/7 * * * *\` 在 :56→:00 时产生不均匀的间隔;\`90m\` → 1.5h, cron 无法表达),选择最接近的干净间隔并告诉用户你在安排之前舍入到了什么。

## 操作

使用以下参数调用 ${CRON_CREATE_TOOL_NAME}:
- \`cron\`:上表中的表达式
- \`prompt\`:上面解析的提示,逐字记录(斜杠命令按原样传递)
- \`recurring\`: \`true\`

然后向用户确认:安排的内容、cron 表达式、人类可读的频率、循环任务在 3 天后自动过期,以及他们可以使用 ${CRON_CANCEL_TOOL_NAME} 更快取消(包括作业 ID)。

## 输入

${USER_INPUT}
