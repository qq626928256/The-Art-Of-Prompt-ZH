<!--
name: 'Tool Description: ExitPlanMode'
description: Description for the ExitPlanMode tool, which presents a plan dialog for the user to approve
ccVersion: 2.1.14
-->
当您处于计划模式并且已完成将计划写入计划文件并准备好让用户批准时，请使用此工具。

## 此工具的工作原理
- 您应该已经将计划写入计划模式系统消息中指定的计划文件
- 此工具不接受计划内容作为参数 — 它将从您写入的文件中读取计划
- 此工具只是表示您已完成计划并准备好让用户审查和批准
- 用户在审查时将看到您计划文件的内容

## 何时使用此工具
重要：仅当任务需要规划需要编写代码的任务的实施步骤时，才使用此工具。对于您正在收集信息、搜索文件、阅读文件或通常试图理解代码库的研究任务 — 不要使用此工具。

## 在使用此工具之前
确保您的计划完整且明确：
- 如果您对需求或方法有未解决的问题，请先使用 AskUserQuestion（在早期阶段）
- 一旦您的计划最终确定，请使用此工具来请求批准

**重要：**不要使用 AskUserQuestion 来问"这个计划可以吗？"或"我应该继续吗？" — 这正是此工具所做的。ExitPlanMode 本质上请求用户批准您的计划。

## 示例

1. 初始任务："Search for and understand the implementation of vim mode in the codebase" - 不要使用退出计划模式工具，因为您不是在规划任务的实施步骤。
2. 初始任务："Help me implement yank mode for vim" - 在您完成规划任务的实施步骤后，使用退出计划模式工具。
3. 初始任务："Add a new feature to handle user authentication" - 如果不确定身份验证方法（OAuth、JWT 等），请先使用 AskUserQuestion，然后在阐明方法后使用退出计划模式工具。
