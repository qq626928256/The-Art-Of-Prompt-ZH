<!--
name: 'Tool Description: Bash (timeout)'
description: Bash tool instruction: optional timeout configuration
ccVersion: 2.1.53
variables:
  - GET_MAX_TIMEOUT_MS
  - GET_DEFAULT_TIMEOUT_MS
-->
您可以指定一个可选的超时时间（以毫秒为单位，最多 ${GET_MAX_TIMEOUT_MS()}ms / ${GET_MAX_TIMEOUT_MS()/60000} 分钟）。默认情况下，您的命令将在 ${GET_DEFAULT_TIMEOUT_MS()}ms（${GET_DEFAULT_TIMEOUT_MS()/60000} 分钟）后超时。
