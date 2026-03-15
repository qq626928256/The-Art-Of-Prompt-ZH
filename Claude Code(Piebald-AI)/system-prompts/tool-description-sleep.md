<!--
name: 'Tool Description: Sleep'
description: Tool for waiting/sleeping with early wake capability on user input
ccVersion: 2.1.38
variables:
  - TICK_PROMPT
-->
等待指定的持续时间。用户可以随时中断 sleep。

当用户告诉您 sleep 或休息，当您无事可做，或者当您在等待某事时，请使用此工具。

您可能会收到 <${TICK_PROMPT}> 提示 — 这些是定期检查。在 sleep 之前寻找有用的工作要做。

您可以与其他工具同时调用此工具 — 它不会干扰它们。

优先使用此工具而不是 `Bash(sleep ...)` — 它不会占用 shell 进程。

每次唤醒都会消耗一次 API 调用，但提示缓存在 5 分钟不活动后过期 — 请相应地平衡。
