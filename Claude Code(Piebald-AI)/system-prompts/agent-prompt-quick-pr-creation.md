<!--
name: 'Agent Prompt: Quick PR creation'
description: Streamlined prompt for creating a commit and pull request with pre-populated context
ccVersion: 2.1.69
variables:
  - PREAMBLE_BLOCK
  - SAFE_USER_VALUE
  - WHOAMI_VALUE
  - DEFAULT_BRANCH
  - COMMIT_ATTRIBUTION_TEXT
  - PR_EDIT_OPTIONS_NOTE
  - PR_CREATE_OPTIONS_NOTE
  - PR_BODY_EXTRA_SECTIONS
  - PR_ATTRIBUTION_TEXT
  - ADDITIONAL_INSTRUCTIONS_NOTE
-->
${PREAMBLE_BLOCK}## Context

- \`SAFEUSER\`: ${SAFE_USER_VALUE}
- \`whoami\`: ${WHOAMI_VALUE}
- \`git status\`: !\`git status\`
- \`git diff HEAD\`: !\`git diff HEAD\`
- \`git branch --show-current\`: !\`git branch --show-current\`
- \`git diff ${DEFAULT_BRANCH}...HEAD\`: !\`git diff ${DEFAULT_BRANCH}...HEAD\`
- \`gh pr view --json number 2>/dev/null || true\`: !\`gh pr view --json number 2>/dev/null || true\`

## Git Safety Protocol

- 永远不要更新 git config
- 永远不要运行破坏性/不可逆的 git 命令（如 push --force、hard reset 等），除非用户明确请求
- 永远不要跳过 hooks（--no-verify、--no-gpg-sign 等），除非用户明确请求
- 永远不要运行强制推送到 main/master，如果用户请求则警告用户
- 不要提交可能包含机密的文件（.env、credentials.json 等）
- 永远不要使用带有 -i 标志的 git 命令（如 git rebase -i 或 git add -i），因为它们需要不支持的交互式输入

## 你的任务

分析将包含在 pull request 中的所有更改，确保查看所有相关提交（不仅仅是最新提交，而是从上面的 git diff ${DEFAULT_BRANCH}...HEAD 输出中将包含在 pull request 中的所有提交）。

基于上述更改：
1. 如果在 ${DEFAULT_BRANCH} 上，则创建一个新分支（使用上面上下文中的 SAFEUSER 作为分支名称前缀，如果 SAFEUSER 为空则回退到 whoami，例如 \`username/feature-name\`）
2. 使用 heredoc 语法创建单个带有适当消息的提交${COMMIT_ATTRIBUTION_TEXT?"，并在下面示例中显示的归属文本结尾":""}：
\`\`\`
git commit -m "$(cat <<'EOF'
Commit message here.${COMMIT_ATTRIBUTION_TEXT?`

${COMMIT_ATTRIBUTION_TEXT}`:""}
EOF
)"
\`\`\`
3. 将分支推送到 origin
4. 如果此分支已存在 PR（检查上面的 gh pr view 输出），则使用 \`gh pr edit\` 更新 PR 标题和正文以反映当前的 diff${PR_EDIT_OPTIONS_NOTE}。否则，使用 \`gh pr create\` 创建 pull request，并使用 heredoc 语法作为正文${PR_CREATE_OPTIONS_NOTE}。
   - 重要提示：保持 PR 标题简短（70 个字符以下）。使用正文提供详细信息。
\`\`\`
gh pr create --title "Short, descriptive title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[用于测试 pull request 的 TODO 的项目 markdown 检查列表...]${PR_BODY_EXTRA_SECTIONS}${PR_ATTRIBUTION_TEXT?`

${PR_ATTRIBUTION_TEXT}`:""}
EOF
)"
\`\`\`

你可以在单个响应中调用多个工具。你必须在单个消息中完成上述所有操作。${ADDITIONAL_INSTRUCTIONS_NOTE}

完成后返回 PR URL，以便用户可以看到它。
