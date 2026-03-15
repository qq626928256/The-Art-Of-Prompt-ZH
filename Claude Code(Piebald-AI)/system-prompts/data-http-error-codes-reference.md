<!--
name: 'Data: HTTP error codes reference'
description: Reference for HTTP error codes returned by the Claude API with common causes and handling strategies
ccVersion: 2.1.73
-->
# HTTP 错误代码参考

此文件记录了 Claude API 返回的 HTTP 错误代码、其常见原因以及如何处理它们。有关特定语言的错误处理示例，请参阅 \`python/\` 或 \`typescript/\` 文件夹。

## 错误代码摘要

| 代码 | 错误类型              | 可重试 | 常见原因                         |
| ---- | ----------------------- | --------- | ------------------------------------ |
| 400  | `invalid_request_error` | 否        | 请求格式或参数无效                 |
| 401  | `authentication_error`  | 否        | API 密钥无效或缺失                 |
| 403  | `permission_error`      | 否        | API 密钥缺乏权限                   |
| 404  | `not_found_error`       | 否        | 端点或模型 ID 无效                 |
| 413  | `request_too_large`     | 否        | 请求超出大小限制                    |
| 429  | `rate_limit_error`      | 是       | 请求过多                            |
| 500  | `api_error`             | 是       | Anthropic 服务问题                  |
| 529  | `overloaded_error`      | 是       | API 暂时过载                        |

## 详细错误信息

### 400 Bad Request

**原因：**

- 请求体中的 JSON 格式错误
- 缺少必需参数（\`model\`、\`max_tokens\`、\`messages\`）
- 参数类型无效（例如，期望整数的位置使用了字符串）
- 消息数组为空
- 消息未交替 user/assistant

**错误示例：**

\`\`\`json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\""
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
\`\`\`

**修复：** 发送前验证请求结构。检查：

- \`model\` 是有效的模型 ID
- \`max_tokens\` 是正整数
- \`messages\` 数组非空且正确交替

---

### 401 Unauthorized

**原因：**

- 缺少 \`x-api-key\` 头部或 \`Authorization\` 头部
- API 密钥格式无效
- API 密钥已吊销或删除

**修复：** 确保正确设置了 \`ANTHROPIC_API_KEY\` 环境变量。

---

### 403 Forbidden

**原因：**

- API 密钥无权访问请求的模型
- 组织级别限制
- 尝试访问 beta 功能但没有 beta 访问权限

**修复：** 在控制台中检查您的 API 密钥权限。您可能需要不同的 API 密钥或请求访问特定功能。

---

### 404 Not Found

**原因：**

- 模型 ID 拼写错误（例如，\`claude-sonnet-4.6\` 而不是 \`claude-sonnet-4-6\`）
- 使用了已弃用的模型 ID
- API 端点无效

**修复：** 使用模型文档中的确切模型 ID。您可以使用别名（例如，\`{{OPUS_ID}}\`）。

---

### 413 Request Too Large

**原因：**

- 请求体超出最大大小
- 输入中的 token 过多
- 图像数据过大

**修复：** 减少输入大小 — 截断对话历史、压缩/调整图像大小，或将大文档分成多个块。
