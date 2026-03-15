<!--
name: 'Data: Message Batches API reference — Python'
description: Python Batches API reference including batch creation, status polling, and result retrieval at 50% cost
ccVersion: 2.1.73
-->
# Message Batches API — Python

Batches API(\`POST /v1/messages/batches\`)以标准价格的 50% 异步处理 Messages API 请求。

## 关键事实

- 每批最多 100,000 个请求或 256 MB
- 大多数批处理在 1 小时内完成;最长 24 小时
- 结果在创建后可保留 29 天
- 所有 token 使用减少 50% 的成本
- 支持所有 Messages API 功能(vision、工具、缓存等)

---

## 创建批处理

\`\`\`python
import anthropic
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id="request-1",
            params=MessageCreateParamsNonStreaming(
                model="{{OPUS_ID}}",
                max_tokens=1024,
                messages=[{"role": "user", "content": "总结气候变化的影响"}]
            )
        ),
        Request(
            custom_id="request-2",
            params=MessageCreateParamsNonStreaming(
                model="{{OPUS_ID}}",
                max_tokens=1024,
                messages=[{"role": "user", "content": "解释量子计算基础"}]
            )
        ),
    ]
)

print(f"批处理 ID: {message_batch.id}")
print(f"状态: {message_batch.processing_status}")
\`\`\`

---

## 轮询完成情况

\`\`\`python
import time

while True:
    batch = client.messages.batches.retrieve(message_batch.id)
    if batch.processing_status == "ended":
        break
    print(f"状态: {batch.processing_status}, 处理中: {batch.request_counts.processing}")
    time.sleep(60)

print("批处理完成!")
print(f"成功: {batch.request_counts.succeeded}")
print(f"错误: {batch.request_counts.errored}")
\`\`\`

---

## 检索结果

> **注意:**下面的示例使用 \`match/case\` 语法,需要 Python 3.10+。对于早期版本,请改用 \`if/elif\` 链。

\`\`\`python
for result in client.messages.batches.results(message_batch.id):
    match result.result.type:
        case "succeeded":
            msg = result.result.message
            text = next((b.text for b in msg.content if b.type == "text"), "")
            print(f"[{result.custom_id}] {text[:100]}")
        case "errored":
            if result.result.error.type == "invalid_request":
                print(f"[{result.custom_id}] 验证错误 - 修复请求并重试")
            else:
                print(f"[{result.custom_id}] 服务器错误 - 可以安全重试")
        case "canceled":
            print(f"[{result.custom_id}] 已取消")
        case "expired":
            print(f"[{result.custom_id}] 已过期 - 重新提交")
\`\`\`

---

## 取消批处理

\`\`\`python
cancelled = client.messages.batches.cancel(message_batch.id)
print(f"状态: {cancelled.processing_status}")  # "canceling"
\`\`\`

---

## 带有提示缓存的批处理

\`\`\`python
shared_system = [
    {"type": "text", "text": "你是一个文学分析师。"},
    {
        "type": "text",
        "text": large_document_text,  # 在所有请求之间共享
        "cache_control": {"type": "ephemeral"}
    }
]

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id=f"analysis-{i}",
            params=MessageCreateParamsNonStreaming(
                model="{{OPUS_ID}}",
                max_tokens=1024,
                system=shared_system,
                messages=[{"role": "user", "content": question}]
            )
        )
        for i, question in enumerate(questions)
    ]
)
\`\`\`

---

## 完整的端到端示例

\`\`\`python
import anthropic
import time
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

# 1. 准备请求
items_to_classify = [
    "产品质量非常好!",
    "糟糕的客户服务,再也不来了。",
    "还可以,没什么特别的。",
]

requests = [
    Request(
        custom_id=f"classify-{i}",
        params=MessageCreateParamsNonStreaming(
            model="{{HAIKU_ID}}",
            max_tokens=50,
            messages=[{
                "role": "user",
                "content": f"分类为正面/负面/中性(一个词): {text}"
            }]
        )
    )
    for i, text in enumerate(items_to_classify)
]

# 2. 创建批处理
batch = client.messages.batches.create(requests=requests)
print(f"已创建批处理: {batch.id}")

# 3. 等待完成
while True:
    batch = client.messages.batches.retrieve(batch.id)
    if batch.processing_status == "ended":
        break
    time.sleep(10)

# 4. 收集结果
results = {}
for result in client.messages.batches.results(batch.id):
    if result.result.type == "succeeded":
        msg = result.result.message
        results[result.custom_id] = next((b.text for b in msg.content if b.type == "text"), "")

for custom_id, classification in sorted(results.items()):
    print(f"{custom_id}: {classification}")
\`\`\`
