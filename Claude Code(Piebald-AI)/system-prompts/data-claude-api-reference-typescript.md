<!--
name: 'Data: Claude API reference — TypeScript'
description: TypeScript SDK reference including installation, client initialization, basic requests, thinking, and multi-turn conversation
ccVersion: 2.1.73
-->
# Claude API — TypeScript

## 安装

```bash
npm install @anthropic-ai/sdk
```

## 客户端初始化

```typescript
import Anthropic from "@anthropic-ai/sdk";

// 默认（使用 ANTHROPIC_API_KEY 环境变量）
const client = new Anthropic();

// 显式 API 密钥
const client = new Anthropic({ apiKey: "your-api-key" });
```

---

## 基本消息请求

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [{ role: "user", content: "What is the capital of France?" }],
});
// response.content 是 ContentBlock[] — 可识别联合。在访问 .text 之前通过 .type 缩小
// （TypeScript 会在没有这个的情况下对 content[0].text 报错）。
for (const block of response.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```

---

## 系统提示

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  system:
    "You are a helpful coding assistant. Always provide examples in Python.",
  messages: [{ role: "user", content: "How do I read a JSON file?" }],
});
```

---

## 视觉（图像）

### URL

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "url", url: "https://example.com/image.png" },
        },
        { type: "text", text: "Describe this image" },
      ],
    },
  ],
});
```

### Base64

```typescript
import fs from "fs";

const imageData = fs.readFileSync("image.png").toString("base64");

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: [
        {
          type: "image",
          source: { type: "base64", media_type: "image/png", data: imageData },
        },
        { type: "text", text: "What's in this image?" },
      ],
    },
  ],
});
```

---

## 提示缓存

### 自动缓存（推荐）

使用顶级 `cache_control` 自动缓存请求中最后一个可缓存块：

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" }, // 自动缓存最后一个可缓存块
  system: "You are an expert on this large document...",
  messages: [{ role: "user", content: "Summarize the key points" }],
});
```

### 手动缓存控制

对于细粒度控制，将 `cache_control` 添加到特定内容块：

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral" }, // 默认 TTL 为 5 分钟
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// 使用显式 TTL（生存时间）
const response2 = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: "You are an expert on this large document...",
      cache_control: { type: "ephemeral", ttl: "1h" }, // 1 小时 TTL
    },
  ],
  messages: [{ role: "user", content: "Summarize the key points" }],
});
```

---

## 扩展思考

> **Opus 4.6 和 Sonnet 4.6：** 使用自适应思考。在 Opus 4.6 和 Sonnet 4.6 上 `budget_tokens` 已弃用。
> **较旧模型：** 使用 `thinking: {type: "enabled", budget_tokens: N}`（必须 < `max_tokens`，最小 1024）。

```typescript
// Opus 4.6：自适应思考（推荐）
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 16000,
  thinking: { type: "adaptive" },
  output_config: { effort: "high" }, // low | medium | high | max
  messages: [
    { role: "user", content: "Solve this math problem step by step..." },
  ],
});

for (const block of response.content) {
  if (block.type === "thinking") {
    console.log("Thinking:", block.thinking);
  } else if (block.type === "text") {
    console.log("Response:", block.text);
  }
}
```

---

## 错误处理

使用 SDK 的类型化异常类 — 永远不要用字符串匹配检查错误消息：

```typescript
import Anthropic from "@anthropic-ai/sdk";

try {
  const response = await client.messages.create({...});
} catch (error) {
  if (error instanceof Anthropic.BadRequestError) {
    console.error("Bad request:", error.message);
  } else if (error instanceof Anthropic.AuthenticationError) {
    console.error("Invalid API key");
  } else if (error instanceof Anthropic.RateLimitError) {
    console.error("Rate limited - retry later");
  } else if (error instanceof Anthropic.APIError) {
    console.error(`API error ${error.status}:`, error.message);
  }
}
```

所有类都扩展 `Anthropic.APIError` 并带有类型化的 `status` 字段。从最具体到最不具体检查。参见[共享/error-codes.md](../../shared/error-codes.md)了解完整的错误代码参考。

---

## 多轮对话

API 是无状态的 — 每次都发送完整的对话历史。使用 `Anthropic.MessageParam[]` 来类型化消息数组：

```typescript
const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "My name is Alice." },
  { role: "assistant", content: "Hello Alice! Nice to meet you." },
  { role: "user", content: "What's my name?" },
];

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: messages,
});
```

**规则：**

- 允许连续的同角色消息 — API 将它们合并为单个轮次
- 第一条消息必须是 `user`
- 对所有 API 数据结构使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Message`、`Anthropic.Tool` 等）— 不要重新定义等效接口

---

### 压缩（长对话）

> **Beta、Opus 4.6 和 Sonnet 4.6。** 当对话接近 200K 上下文窗口时，压缩会自动在服务端总结较早的上下文。API 返回 `compaction` 块；您必须在后续请求中将其传回 — 附加 `response.content`，而不仅仅是文本。

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const messages: Anthropic.Beta.BetaMessageParam[] = [];

async function chat(userMessage: string): Promise<string> {
  messages.push({ role: "user", content: userMessage });

  const response = await client.beta.messages.create({
    betas: ["compact-2026-01-12"],
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    messages,
    context_management: {
      edits: [{ type: "compact_20260112" }],
    },
  });

  // 附加完整内容 — 必须保留压缩块
  messages.push({ role: "assistant", content: response.content });

  const textBlock = response.content.find(
    (b): b is Anthropic.Beta.BetaTextBlock => b.type === "text",
  );
  return textBlock?.text ?? "";
}

// 当上下文变大时自动触发压缩
console.log(await chat("Help me build a Python web scraper"));
console.log(await chat("Add support for JavaScript-rendered pages"));
console.log(await chat("Now add rate limiting and error handling"));
```

---

## 停止原因

响应中的 `stop_reason` 字段指示模型为何停止生成：

| 值 | 含义 |
|-------|---------|
| `end_turn` | Claude 自然完成了响应 |
| `max_tokens` | 达到 `max_tokens` 限制 — 增加它或使用流式传输 |
| `stop_sequence` | 达到自定义停止序列 |
| `tool_use` | Claude 想要调用工具 — 执行它并继续 |
| `pause_turn` | 模型暂停，可以恢复（agentic 流程） |
| `refusal` | Claude 因安全原因拒绝 — 输出可能不匹配 schema |

---

## 成本优化策略

### 1. 对重复上下文使用提示缓存

```typescript
// 自动缓存（最简单 — 缓存最后一个可缓存块）
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" },
  system: largeDocumentText, // 例如，50KB 上下文
  messages: [{ role: "user", content: "Summarize the key points" }],
});

// 第一次请求：全价
// 后续请求：缓存部分约便宜 90%
```

### 2. 在请求前使用 Token 计数

```typescript
const countResponse = await client.messages.countTokens({
  model: "{{OPUS_ID}}",
  messages: messages,
  system: system,
});

const estimatedInputCost = countResponse.input_tokens * 0.000005; // $5/1M tokens
console.log(`Estimated input cost: $${estimatedInputCost.toFixed(4)}`);
```
