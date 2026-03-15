<!--
name: 'Data: Claude API reference — cURL'
description: Raw API reference for Claude API for use with cURL or else Raw HTTP
ccVersion: 2.1.73
-->
# Claude API — cURL / Raw HTTP

当用户需要原始 HTTP 请求或使用没有官方 SDK 的语言时，请使用这些示例。

## 设置

\`\`\`bash
export ANTHROPIC_API_KEY="your-api-key"
\`\`\`

---

## 基本消息请求

\`\`\`bash
curl https://api.anthropic.com/v1/messages \\
  -H "Content-Type: application/json" \\
  -H "x-api-key: $ANTHROPIC_API_KEY" \\
  -H "anthropic-version: 2023-06-01" \\
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 1024,
    "messages": [
      {"role": "user", "content": "法国的首都是什么？"}
    ]
  }'
\`\`\`

### 解析响应

使用 \`jq\` 从 JSON 响应中提取字段。不要使用 \`grep\`/\`sed\` —
JSON 字符串可以包含任何字符，正则表达式解析会在引号、
转义符或多行内容上中断。

\`\`\`bash
# 捕获响应，然后提取字段
response=$(curl -s https://api.anthropic.com/v1/messages \\
  -H "Content-Type: application/json" \\
  -H "x-api-key: $ANTHROPIC_API_KEY" \\
  -H "anthropic-version: 2023-06-01" \\
  -d '{"model":"{{OPUS_ID}}","max_tokens":1024,"messages":[{"role":"user","content":"你好"}]}')

# 打印第一个文本块（-r 去除 JSON 引号）
echo "$response" | jq -r '.content[0].text'

# 读取使用字段
input_tokens=$(echo "$response" | jq -r '.usage.input_tokens')
output_tokens=$(echo "$response" | jq -r '.usage.output_tokens')

# 读取停止原因（用于工具使用循环）
stop_reason=$(echo "$response" | jq -r '.stop_reason')

# 提取所有文本块（content 是一个数组；过滤到 type=="text"）
echo "$response" | jq -r '.content[] | select(.type == "text") | .text'
\`\`\`


---

## 流式传输（SSE）

\`\`\`bash
curl https://api.anthropic.com/v1/messages \\
  -H "Content-Type: application/json" \\
  -H "x-api-key: $ANTHROPIC_API_KEY" \\
  -H "anthropic-version: 2023-06-01" \\
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 1024,
    "stream": true,
    "messages": [{"role": "user", "content": "写一首俳句"}]
  }'
\`\`\`

响应是服务器发送事件流：

\`\`\`
event: message_start
data: {"type":"message_start","message":{"id":"msg_...","type":"message",...}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"你好"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn"},"usage":{"output_tokens":12}}

event: message_stop
data: {"type":"message_stop"}
\`\`\`

---

## 工具使用

\`\`\`bash
curl https://api.anthropic.com/v1/messages \\
  -H "Content-Type: application/json" \\
  -H "x-api-key: $ANTHROPIC_API_KEY" \\
  -H "anthropic-version: 2023-06-01" \\
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 1024,
    "tools": [{
      "name": "get_weather",
      "description": "获取给定位置的当前天气",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "城市名称"}
        },
        "required": ["location"]
      }
    }],
    "messages": [{"role": "user", "content": "巴黎的天气如何？"}]
  }'
\`\`\`

当 Claude 使用 \`tool_use\` 块响应时，将结果发送回去：

\`\`\`bash
curl https://api.anthropic.com/v1/messages \\
  -H "Content-Type: application/json" \\
  -H "x-api-key: $ANTHROPIC_API_KEY" \\
  -H "anthropic-version: 2023-06-01" \\
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 1024,
    "tools": [{
      "name": "get_weather",
      "description": "获取给定位置的当前天气",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "城市名称"}
        },
        "required": ["location"]
      }
    }],
    "messages": [
      {"role": "user", "content": "巴黎的天气如何？"},
      {"role": "assistant", "content": [
        {"type": "text", "text": "让我检查天气。"},
        {"type": "tool_use", "id": "toolu_abc123", "name": "get_weather", "input": {"location": "巴黎"}}
      ]},
      {"role": "user", "content": [
        {"type": "tool_result", "tool_use_id": "toolu_abc123", "content": "72°F 和晴天"}
      ]}
    ]
  }'
\`\`\`

---

## 扩展思考

> **Opus 4.6 和 Sonnet 4.6：** 使用自适应思考。\`budget_tokens\` 在 Opus 4.6 和 Sonnet 4.6 上都已弃用。
> **较旧的模型：** 使用 \`"type": "enabled"\` 和 \`"budget_tokens": N\`（必须 < \`max_tokens\`，最小 1024）。

\`\`\`bash
# Opus 4.6：自适应思考（推荐）
curl https://api.anthropic.com/v1/messages \\
  -H "Content-Type: application/json" \\
  -H "x-api-key: $ANTHROPIC_API_KEY" \\
  -H "anthropic-version: 2023-06-01" \\
  -d '{
    "model": "{{OPUS_ID}}",
    "max_tokens": 16000,
    "thinking": {
      "type": "adaptive"
    },
    "output_config": {
      "effort": "high"
    },
    "messages": [{"role": "user", "content": "逐步解决这个问题..."}]
  }'
\`\`\`

---

## 必需的标头

| 标头              | 值              | 描述                |
| ------------------- | ------------------ | -------------------------- |
| \`Content-Type\`      | \`application/json\` | 必需                   |
| \`x-api-key\`         | 您的 API 密钥       | 身份验证             |
| \`anthropic-version\` | \`2023-06-01\`       | API 版本                |
| \`anthropic-beta\`    | Beta 功能 ID   | Beta 功能所需 |
