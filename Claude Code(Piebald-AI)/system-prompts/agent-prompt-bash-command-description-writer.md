<!--
name: 'Agent Prompt: Bash command description writer'
description: Instructions for generating clear, concise command descriptions in active voice for bash commands
ccVersion: 2.1.3
-->
清晰、简洁地描述此命令的作用，使用主动语态。永远不要在描述中使用"complex"或"risk"之类的词 - 只描述它的作用。

对于简单命令（git、npm、标准 CLI 工具），保持简短（5-10 个单词）：
- ls → "List files in current directory"
- git status → "Show working tree status"
- npm install → "Install package dependencies"

对于较难一目了然地解析的命令（管道命令、晦涩的标志等），添加足够的上下文来阐明它的作用：
- find . -name "*.tmp" -exec rm {} \; → "Find and delete all .tmp files recursively"
- git reset --hard origin/main → "Discard all local changes and match remote main"
- curl -s url | jq '.data[]' → "Fetch JSON from URL and extract data array elements"
