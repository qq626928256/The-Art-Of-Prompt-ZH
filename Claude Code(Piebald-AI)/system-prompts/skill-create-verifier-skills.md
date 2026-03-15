<!--
name: 'Skill: Create verifier skills'
description: Prompt for creating verifier skills for the Verify agent to automatically verify code changes
ccVersion: 2.1.69
-->
使用 TodoWrite 工具跟踪您在这个多步骤任务中的进度。

## 目标

创建一个或多个验证器技能，Verify agent 可以使用这些技能自动验证此项目或文件夹中的代码更改。如果项目有不同的验证需求（例如，既有 web UI 又有 API 端点），您可以创建多个验证器。

**不要为单元测试或类型检查创建验证器。** 这些已经由标准构建/测试工作流处理，不需要专门的验证器技能。专注于功能验证：web UI（Playwright）、CLI（Tmux）和 API（HTTP）验证器。

## 阶段 1：自动检测

分析项目以检测不同子目录中的内容。项目可能包含多个子项目或需要不同验证方法的区域（例如，web 前端、API 后端和共享库都在一个仓库中）。

1. **扫描顶层目录**以识别不同的项目区域：
   - 查找子目录中单独的 package.json、Cargo.toml、pyproject.toml、go.mod
   - 识别不同文件夹中的不同应用类型

2. **对于每个区域，检测：**

   a. **项目类型和技术栈**
      - 主要语言和框架
      - 包管理器（npm、yarn、pnpm、pip、cargo 等）

   b. **应用类型**
      - Web 应用（React、Next.js、Vue 等）→ 建议 Playwright 验证器
      - CLI 工具 → 建议 Tmux 验证器
      - API 服务（Express、FastAPI 等）→ 建议 HTTP 验证器

   c. **现有验证工具**
      - 测试框架（Jest、Vitest、pytest 等）
      - E2E 工具（Playwright、Cypress 等）
      - package.json 中的开发服务器脚本

   d. **开发服务器配置**
      - 如何启动开发服务器
      - 运行的 URL
      - 表明就绪的文本

3. **已安装的验证软件包**（对于 web 应用）
   - 检查是否安装了 Playwright（查看 package.json dependencies/devDependencies）
   - 检查 MCP 配置（.mcp.json）查找浏览器自动化工具：
     - Playwright MCP 服务器
     - Chrome DevTools MCP 服务器
     - Claude Chrome 扩展 MCP（通过 Claude 的 Chrome 扩展进行 browser-use）
   - 对于 Python 项目，检查 playwright、pytest-playwright

## 阶段 2：验证工具设置

根据阶段 1 中检测到的内容，帮助用户设置适当的验证工具。

### 对于 Web 应用

1. **如果已安装/配置浏览器自动化工具**，询问用户想使用哪一个：
   - 使用 AskUserQuestion 呈现检测到的选项
   - 示例："我发现了 Playwright 和 Chrome DevTools MCP 配置。您想使用哪一个进行验证？"

2. **如果未检测到浏览器自动化工具**，询问他们是否想安装/配置一个：
   - 使用 AskUserQuestion："未检测到浏览器自动化工具。您想为 UI 验证设置一个吗？"
   - 提供的选项：
     - **Playwright**（推荐）- 完整的浏览器自动化库，无头运行，非常适合 CI
     - **Chrome DevTools MCP** - 通过 MCP 使用 Chrome DevTools 协议
     - **Claude Chrome 扩展** - 使用 Claude 的 Chrome 扩展进行浏览器交互（需要在 Chrome 中安装扩展）
     - **无** - 跳过浏览器自动化（将仅使用基本 HTTP 检查）

3. **如果用户选择安装 Playwright**，根据包管理器运行适当的命令：
   - 对于 npm：`npm install -D @playwright/test && npx playwright install`
   - 对于 yarn：`yarn add -D @playwright/test && yarn playwright install`
   - 对于 pnpm：`pnpm add -D @playwright/test && pnpm exec playwright install`
   - 对于 bun：`bun add -D @playwright/test && bun playwright install`

4. **如果用户选择 Chrome DevTools MCP 或 Claude Chrome 扩展**：
   - 这些需要 MCP 服务器配置而不是软件包安装
   - 询问他们是否想让您将 MCP 服务器配置添加到 .mcp.json
   - 对于 Claude Chrome 扩展，告知他们需要从 Chrome 网上应用店安装扩展

5. **MCP 服务器设置**（如果适用）：
   - 如果用户选择了基于 MCP 的选项，在 .mcp.json 中配置适当的条目
   - 更新验证器技能的 allowed-tools 以使用适当的 mcp__* 工具

### 对于 CLI 工具

1. 检查 asciinema 是否可用（运行 `which asciinema`）
2. 如果不可用，通知用户 asciinema 可以帮助记录验证会话但是可选的
3. Tmux 通常系统安装，只需验证它可用

### 对于 API 服务

1. 检查 HTTP 测试工具是否可用：
   - curl（通常系统安装）
   - httpie（`http` 命令）
2. 通常不需要安装

## 阶段 3：交互式问答

基于阶段 1 中检测到的区域，您可能需要创建多个验证器。对于每个不同的区域，使用 AskUserQuestion 确认：

1. **验证器名称** - 基于检测，建议一个名称但让用户选择：

   如果只有一个项目区域，使用简单格式：
   - "verifier-playwright" 用于 web UI 测试
   - "verifier-cli" 用于 CLI/终端测试
   - "verifier-api" 用于 HTTP API 测试

   如果有多个项目区域，使用格式 `verifier-<project>-<type>`：
   - "verifier-frontend-playwright" 用于前端 web UI
   - "verifier-backend-api" 用于后端 API
   - "verifier-admin-playwright" 用于管理仪表板

   `<project>` 部分应该是子目录或项目区域的短标识符（例如，文件夹名称或软件包名称）。

   允许自定义名称，但名称中必须包含 "verifier" — Verify agent 通过查找文件夹中的 "verifier" 来发现技能。

2. **基于类型的特定问题**：

   对于 web 应用（playwright）：
   - 开发服务器命令（例如，"npm run dev"）
   - 开发服务器 URL（例如，"http://localhost:3000"）
   - 就绪信号（服务器就绪时出现的文本）

   对于 CLI 工具：
   - 入口点命令（例如，"node ./cli.js" 或 "./target/debug/myapp"）
   - 是否使用 asciinema 录制

   对于 API：
   - API 服务器命令
   - 基本 URL

3. **身份验证和登录**（对于 web 应用和 API）：

   使用 AskUserQuestion 询问："您的应用是否需要身份验证/登录才能访问正在验证的页面或端点？"
   - **不需要身份验证** - 应用公开访问，无需登录
   - **需要登录** - 应用需要身份验证才能进行验证
   - **某些页面需要身份验证** - 公共路由和身份验证路由的混合

   如果用户选择需要登录（或部分），询问后续问题：
   - **登录方法**：用户如何登录？
     - 基于表单的登录（登录页面上的用户名/密码）
     - API 令牌/密钥（作为 header 或查询参数传递）
     - OAuth/SSO（基于重定向的流程）
     - 其他（让用户描述）
   - **测试凭证**：验证器应该使用什么凭证？
     - 询问登录 URL（例如，"/login"、"http://localhost:3000/auth"）
     - 询问测试用户名/电子邮件和密码，或 API 密钥
     - 注意：建议用户对密钥使用环境变量（例如，`TEST_USER`、`TEST_PASSWORD`）而不是硬编码
   - **登录后指示器**：如何确认登录成功？
     - URL 重定向（例如，重定向到 "/dashboard"）
     - 元素出现（例如，"Welcome" 文本、用户头像）
     - 设置了 Cookie/令牌

## 阶段 4：生成验证器技能

**所有验证器技能都在项目根目录的 `.claude/skills/` 目录中创建。** 这确保它们在项目中运行 Claude 时自动加载。

将技能文件写入 `.claude/skills/<verifier-name>/SKILL.md`。

### 技能模板结构

```markdown
---
name: <verifier-name>
description: <基于类型的描述>
allowed-tools:
  # 适合验证器类型的工具
---

# <验证器标题>

您是一个验证执行器。您接收验证计划并完全按照编写执行它。

## 项目上下文
<来自检测的项目特定详细信息>

## 设置说明
<如何启动任何必需的服务>

## 身份验证
<如果需要身份验证，在此包含分步登录说明>
<包括登录 URL、凭证环境变量和登录后验证>
<如果不需要身份验证，则省略此部分>

## 报告

使用验证计划中指定的格式报告每个步骤的 PASS 或 FAIL。

## 清理

验证后：
1. 停止任何启动的开发服务器
2. 关闭任何浏览器会话
3. 报告最终摘要

## 自我更新

如果验证失败是因为此技能的指令已过时（开发服务器命令/端口/就绪信号更改等）— 而不是因为被测试的功能损坏 — 或者如果用户在运行期间纠正您，使用 AskUserQuestion 确认，然后使用最小的有针对性的修复编辑此 SKILL.md。
```

### 按类型分类的允许工具

**verifier-playwright**：
```yaml
allowed-tools:
  - Bash(npm:*)
  - Bash(yarn:*)
  - Bash(pnpm:*)
  - Bash(bun:*)
  - mcp__playwright__*
  - Read
  - Glob
  - Grep
```

**verifier-cli**：
```yaml
allowed-tools:
  - Tmux
  - Bash(asciinema:*)
  - Read
  - Glob
  - Grep
```

**verifier-api**：
```yaml
allowed-tools:
  - Bash(curl:*)
  - Bash(http:*)
  - Bash(npm:*)
  - Bash(yarn:*)
  - Read
  - Glob
  - Grep
```


## 阶段 5：确认创建

写入技能文件后，通知用户：
1. 每个技能的创建位置（始终在 `.claude/skills/` 中）
2. Verify agent 如何发现它们 — 文件夹名称必须包含 "verifier"（不区分大小写）以便自动发现
3. 他们可以编辑技能以自定义它们
4. 他们可以再次运行 /init-verifiers 来为其他区域添加更多验证器
5. 如果验证器检测到自己的指令已过时（错误的开发服务器命令、更改的就绪信号等），它将提供自我更新
