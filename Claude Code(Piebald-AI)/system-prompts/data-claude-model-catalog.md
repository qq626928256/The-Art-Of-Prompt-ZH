<!--
name: 'Data: Claude model catalog'
description: Catalog of current and legacy Claude models with exact model IDs, aliases, context windows, and pricing
ccVersion: 2.1.73
-->
# Claude 模型目录

**仅使用此文件中列出的确切模型 ID。** 永远不要猜测或构造模型 ID — 错误的 ID 将导致 API 错误。尽可能使用别名。有关最新信息，请使用 WebFetch 获取 \`shared/live-sources.md\` 中的模型概述 URL。

## 当前模型（推荐）

| 友好名称        | 别名（使用此列）     | 完整 ID                       | 上下文         | 最大输出 | 状态   |
| ---------------- | -------------------- | ----------------------------- | -------------- | -------- | ------ |
| Claude Opus 4.6  | \`claude-opus-4-6\`    | —                             | 200K (1M beta) | 128K     | Active |
| Claude Sonnet 4.6 | \`claude-sonnet-4-6\` | -                             | 200K (1M beta) | 64K      | Active |
| Claude Haiku 4.5  | \`claude-haiku-4-5\`  | \`claude-haiku-4-5-20251001\`   | 200K           | 64K      | Active |

### 模型描述

- **Claude Opus 4.6** — 我们最智能的模型，用于构建 Agent 和编码。支持自适应思考（推荐），128K 最大输出 token（大输出需要流式传输）。通过 \`context-1m-2025-08-07\` 头部提供 beta 版 1M 上下文窗口。
- **Claude Sonnet 4.6** — 速度和智能的最佳组合。支持自适应思考（推荐）。通过 \`context-1m-2025-08-07\` 头部提供 beta 版 1M 上下文窗口。64K 最大输出 token。
- **Claude Haiku 4.5** — 最快且最具成本效益的模型，用于简单任务。

## 遗留模型（仍活跃）

| 友好名称        | 别名（使用此列）     | 完整 ID                       | 状态   |
| ---------------- | -------------------- | ----------------------------- | ------ |
| Claude Opus 4.5  | \`claude-opus-4-5\`   | \`claude-opus-4-5-20251101\`    | Active |
| Claude Opus 4.1  | \`claude-opus-4-1\`   | \`claude-opus-4-1-20250805\`    | Active |
| Claude Sonnet 4.5 | \`claude-sonnet-4-5\` | \`claude-sonnet-4-5-20250929\`  | Active |
| Claude Sonnet 4  | \`claude-sonnet-4-0\` | \`claude-sonnet-4-20250514\`    | Active |
| Claude Opus 4    | \`claude-opus-4-0\`   | \`claude-opus-4-20250514\`      | Active |

## 已弃用模型（即将停用）

| 友好名称     | 别名（使用此列） | 完整 ID                       | 状态       | 停用日期     |
| ---------------- | ------------------- | ----------------------------- | ---------- | ------------ |
| Claude Haiku 3    | —                   | \`claude-3-haiku-20240307\`     | Deprecated | Apr 19, 2026 |

## 已停用模型（不再可用）

| 友好名称          | 完整 ID                       | 停用日期     |
| ------------------- | ----------------------------- | ------------- |
| Claude Sonnet 3.7  | \`claude-3-7-sonnet-20250219\`  | Feb 19, 2026 |
| Claude Haiku 3.5   | \`claude-3-5-haiku-20241022\`   | Feb 19, 2026 |
| Claude Opus 3      | \`claude-3-opus-20240229\`      | Jan 5, 2026  |
| Claude Sonnet 3.5  | \`claude-3-5-sonnet-20241022\`  | Oct 28, 2025 |
| Claude Sonnet 3.5  | \`claude-3-5-sonnet-20240620\`  | Oct 28, 2025 |
| Claude Sonnet 3    | \`claude-3-sonnet-20240229\`    | Jul 21, 2025 |
| Claude 2.1         | \`claude-2.1\`                  | Jul 21, 2025 |
| Claude 2.0         | \`claude-2.0\`                  | Jul 21, 2025 |

## 解析用户请求

当用户按名称请求模型时，使用此表查找正确的模型 ID：

| 用户说...                                 | 使用此模型 ID              |
| ----------------------------------------- | -------------------------- |
| "opus", "most powerful"                   | \`claude-opus-4-6\`         |
| "opus 4.6"                                | \`claude-opus-4-6\`         |
| "opus 4.5"                                | \`claude-opus-4-5\`         |
| "opus 4.1"                                | \`claude-opus-4-1\`         |
| "opus 4", "opus 4.0"                      | \`claude-opus-4-0\`         |
| "sonnet", "balanced"                      | \`claude-sonnet-4-6\`       |
| "sonnet 4.6"                              | \`claude-sonnet-4-6\`       |
| "sonnet 4.5"                              | \`claude-sonnet-4-5\`       |
| "sonnet 4", "sonnet 4.0"                  | \`claude-sonnet-4-0\`       |
| "sonnet 3.7"                              | 已停用 — 建议 \`claude-sonnet-4-5\` |
| "sonnet 3.5"                              | 已停用 — 建议 \`claude-sonnet-4-5\` |
| "haiku", "fast", "cheap"                  | \`claude-haiku-4-5\`        |
| "haiku 4.5"                               | \`claude-haiku-4-5\`        |
| "haiku 3.5"                               | 已停用 — 建议 \`claude-haiku-4-5\` |
| "haiku 3"                                 | 已弃用 — 建议 \`claude-haiku-4-5\` |
