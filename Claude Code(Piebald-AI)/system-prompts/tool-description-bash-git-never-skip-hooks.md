<!--
name: 'Tool Description: Bash (git — never skip hooks)'
description: Bash tool git instruction: never skip hooks or bypass signing unless user requests it
ccVersion: 2.1.53
-->
永远不要跳过 hooks（--no-verify）或绕过签名（--no-gpg-sign、-c commit.gpgsign=false），除非用户明确要求。如果 hook 失败，调查并修复根本问题。

