<!--
name: 'Tool Description: Bash (sandbox — tmpdir)'
description: Use $TMPDIR for temporary files in sandbox mode
ccVersion: 2.1.53
variables:
  - SANDBOX_TMPDIR_FN
-->
对于临时文件，始终使用 \`$TMPDIR\` 环境变量（或 \`${SANDBOX_TMPDIR_FN()}\` 作为后备）。在沙箱模式下，TMPDIR 会自动设置为正确的沙箱可写目录。不要直接使用 \`/tmp\` —— 改用 \`$TMPDIR\` 或 \`${SANDBOX_TMPDIR_FN()}\`。

