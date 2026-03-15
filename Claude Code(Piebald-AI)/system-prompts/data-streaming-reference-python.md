<!--
name: 'Data: Streaming reference — Python'
description: Python streaming reference including sync/async streaming and handling different content types
ccVersion: 2.1.63
-->
# Streaming — Python

## 快速开始

\`\`\`python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "写一个故事"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
\`\`\`

### 异步

\`\`\`python
async with async_client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "写一个故事"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
\`\`\`

---

## 处理不同的内容类型

Claude 可能返回文本、思考块或工具使用。适当地处理每种情况:

> **Opus 4.6:** 使用 \`thinking: {type: "adaptive"}\`。对于旧模型,使用 \`thinking: {type: "enabled", budget_tokens: N}\` 代替。

\`\`\`python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "分析这个问题"}]
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            if event.content_block.type == "thinking":
                print("\\n[思考中...]")
            elif event.content_block.type == "text":
                print("\\n[响应:]")

        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
\`\`\`

---

## 使用工具的流式传输

Python 工具运行器目前返回完整的消息。如果需要在手动循环中逐个 token 流式传输,请对单个 API 调用使用流式传输:

\`\`\`python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    tools=tools,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    response = stream.get_final_message()
    # 如果 response.stop_reason == "tool_use" 则继续执行工具
\`\`\`

---

## 获取最终消息

\`\`\`python
with client.messages.stream(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "你好"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    # 流式传输后获取完整消息
    final_message = stream.get_final_message()
    print(f"\\n\\n使用的 token: {final_message.usage.output_tokens}")
\`\`\`

---

## 带进度更新的流式传输

\`\`\`python
def stream_with_progress(client, **kwargs):
    """带进度更新的流式响应。"""
    total_tokens = 0
    content_parts = []

    with client.messages.stream(**kwargs) as stream:
        for event in stream:
            if event.type == "content_block_delta":
                if event.delta.type == "text_delta":
                    text = event.delta.text
                    content_parts.append(text)
                    print(text, end="", flush=True)

            elif event.type == "message_delta":
                if event.usage and event.usage.output_tokens is not None:
                    total_tokens = event.usage.output_tokens

        final_message = stream.get_final_message()

    print(f"\\n\\n[使用的 token: {total_tokens}]")
    return "".join(content_parts)
\`\`\`

---

## 流中的错误处理

\`\`\`python
try:
    with client.messages.stream(
        model="{{OPUS_ID}}",
        max_tokens=1024,
        messages=[{"role": "user", "content": "写一个故事"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
except anthropic.APIConnectionError:
    print("\\n连接丢失。请重试。")
except anthropic.RateLimitError:
    print("\\n速率限制。请等待并重试。")
except anthropic.APIStatusError as e:
    print(f"\\nAPI 错误: {e.status_code}")
\`\`\`

---

## 流事件类型

| 事件类型              | 描述                 | 触发时间                     |
| --------------------- | -------------------- | ---------------------------- |
| \`message_start\`       | 包含消息元数据       | 开始时触发一次               |
| \`content_block_start\` | 新内容块开始         | 当文本/tool_use 块开始时     |
| \`content_block_delta\` | 增量内容更新         | 每个 token/chunk              |
| \`content_block_stop\`  | 内容块完成           | 当块完成时                   |
| \`message_delta\`       | 消息级更新           | 包含 \`stop_reason\`, usage |
| \`message_stop\`        | 消息完成             | 结束时触发一次               |

## 最佳实践

1. **始终刷新输出** — 使用 \`flush=True\` 立即显示 token
2. **处理部分响应** — 如果流被中断,你可能有不完整的内容
3. **跟踪 token 使用** — \`message_delta\` 事件包含使用信息
4. **使用超时** — 为你的应用设置适当的超时
5. **默认使用流式传输** — 即使使用流式传输也使用 \`.get_final_message()\` 获取完整响应,这给你超时保护而无需处理单个事件
