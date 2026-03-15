<!--
name: 'Data: Tool use reference — TypeScript'
description: TypeScript tool use reference including tool runner, manual agentic loop, code execution, and structured outputs
ccVersion: 2.1.73
-->
# 工具使用 — TypeScript

有关概念概述（工具定义、工具选择、技巧），请参阅 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## 工具运行器（推荐）

**Beta：** TypeScript SDK 中的工具运行器处于 beta 阶段。

使用 `betaZodTool` 和 Zod schema 定义带有 `run` 函数的工具，然后将它们传递给 `client.beta.messages.toolRunner()`：

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";

const client = new Anthropic();

const getWeather = betaZodTool({
  name: "get_weather",
  description: "Get current weather for a location",
  inputSchema: z.object({
    location: z.string().describe("City and state, e.g., San Francisco, CA"),
    unit: z.enum(["celsius", "fahrenheit"]).optional(),
  }),
  run: async (input) => {
    // 您的实现在这里
    return `72°F and sunny in ${input.location}`;
  },
});

// 工具运行器处理 agentic 循环并返回最终消息
const finalMessage = await client.beta.messages.toolRunner({
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  tools: [getWeather],
  messages: [{ role: "user", content: "What's the weather in Paris?" }],
});

console.log(finalMessage.content);
```

**工具运行器的主要好处：**

- 无需手动循环 — SDK 处理调用工具和反馈结果
- 通过 Zod schema 实现类型安全的工具输入
- 工具 schema 从 Zod 定义自动生成
- 当 Claude 没有更多工具调用时迭代自动停止

---

## 手动 Agentic 循环

当您需要细粒度控制时使用此方法（自定义日志记录、条件工具执行、流式传输单个迭代、人工批准循环）：

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const tools: Anthropic.Tool[] = [...]; // 您的工具定义
let messages: Anthropic.MessageParam[] = [{ role: "user", content: userInput }];

while (true) {
  const response = await client.messages.create({
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    tools: tools,
    messages: messages,
  });

  if (response.stop_reason === "end_turn") break;

  // 服务端工具达到迭代限制；附加助手轮次并重新发送以继续
  if (response.stop_reason === "pause_turn") {
    messages.push({ role: "assistant", content: response.content });
    continue;
  }

  const toolUseBlocks = response.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use",
  );

  messages.push({ role: "assistant", content: response.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    const result = await executeTool(tool.name, tool.input);
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: result,
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

### 流式传输手动循环

在需要在手动循环中进行流式传输时，使用 `client.messages.stream()` + `finalMessage()` 而不是 `.create()`。文本增量在每次迭代中流式传输；`finalMessage()` 收集完整的 `Message`，以便您可以检查 `stop_reason` 并提取工具使用块：

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const tools: Anthropic.Tool[] = [...];
let messages: Anthropic.MessageParam[] = [{ role: "user", content: userInput }];

while (true) {
  const stream = client.messages.stream({
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    tools,
    messages,
  });

  // 在每次迭代中流式传输文本增量
  stream.on("text", (delta) => {
    process.stdout.write(delta);
  });

  // finalMessage() 解析为完整的 Message — 无需
  // 手动连接 .on("message") / .on("error") / .on("abort")
  const message = await stream.finalMessage();

  if (message.stop_reason === "end_turn") break;

  // 服务端工具达到迭代限制；附加助手轮次并重新发送以继续
  if (message.stop_reason === "pause_turn") {
    messages.push({ role: "assistant", content: message.content });
    continue;
  }

  const toolUseBlocks = message.content.filter(
    (b): b is Anthropic.ToolUseBlock => b.type === "tool_use",
  );

  messages.push({ role: "assistant", content: message.content });

  const toolResults: Anthropic.ToolResultBlockParam[] = [];
  for (const tool of toolUseBlocks) {
    const result = await executeTool(tool.name, tool.input);
    toolResults.push({
      type: "tool_result",
      tool_use_id: tool.id,
      content: result,
    });
  }

  messages.push({ role: "user", content: toolResults });
}
```

> **重要：** 不要将 `.on()` 事件包装在 `new Promise()` 中以收集最终消息 — 而是使用 `stream.finalMessage()`。SDK 在内部处理所有错误/中止/完成状态。

> **循环中的错误处理：** 使用 SDK 的类型化异常（例如，`Anthropic.RateLimitError`、`Anthropic.APIError`）— 参见[错误处理](./README.md#error-handling)了解示例。不要用字符串匹配检查错误消息。

> **SDK 类型：** 对所有 API 相关的数据结构使用 `Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.ToolUseBlock`、`Anthropic.ToolResultBlockParam`、`Anthropic.Message` 等。不要重新定义等效接口。

---

## 处理工具结果

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  tools: tools,
  messages: [{ role: "user", content: "What's the weather in Paris?" }],
});

for (const block of response.content) {
  if (block.type === "tool_use") {
    const result = await executeTool(block.name, block.input);

    const followup = await client.messages.create({
      model: "{{OPUS_ID}}",
      max_tokens: 1024,
      tools: tools,
      messages: [
        { role: "user", content: "What's the weather in Paris?" },
        { role: "assistant", content: response.content },
        {
          role: "user",
          content: [
            { type: "tool_result", tool_use_id: block.id, content: result },
          ],
        },
      ],
    });
  }
}
```

---

## 工具选择

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  tools: tools,
  tool_choice: { type: "tool", name: "get_weather" },
  messages: [{ role: "user", content: "What's the weather in Paris?" }],
});
```

---

## 服务端工具

带版本后缀的 `type` 字符串；`name` 按接口固定。传递普通对象字面量 — `ToolUnion` 类型在结构上满足。**`name`/`type` 对必须匹配接口**：混合 `str_replace_based_edit_tool`（20250728 名称）与 `text_editor_20250124`（期望 `str_replace_editor`）会导致 TS2322 错误。

**不要类型注释为 `Tool[]`** — `Tool` 只是自定义工具变体。让结构类型从 `tools` 参数推断，或者如果必须，注释为 `Anthropic.Messages.ToolUnion[]`：

```typescript
// ✓ 让推断工作 — 无注释
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  tools: [
    { type: "text_editor_20250728", name: "str_replace_based_edit_tool" },
    { type: "bash_20250124", name: "bash" },
    { type: "web_search_20260209", name: "web_search" },
    { type: "code_execution_20260120", name: "code_execution" },
  ],
  messages: [{ role: "user", content: "..." }],
});

// ✗ 这是 TS2352 — Tool 只是 CUSTOM 工具变体
// const tools: Anthropic.Tool[] = [{ type: "text_editor_20250728", ... }]
```

| 接口 | `name` | `type` |
|---|---|---|
| `ToolTextEditor20250124` | `str_replace_editor` | `text_editor_20250124` |
| `ToolTextEditor20250429` | `str_replace_based_edit_tool` | `text_editor_20250429` |
| `ToolTextEditor20250728` | `str_replace_based_edit_tool` | `text_editor_20250728` |
| `ToolBash20250124` | `bash` | `bash_20250124` |
| `WebSearchTool20260209` | `web_search` | `web_search_20260209` |
| `WebFetchTool20260209` | `web_fetch` | `web_fetch_20260209` |
| `CodeExecutionTool20260120` | `code_execution` | `code_execution_20260120` |

**不要混合 beta 和非 beta 类型**：如果您调用 `client.beta.messages.create()`，响应 `content` 是 `BetaContentBlock[]` — 您不能将其传递给非 beta `ContentBlockParam[]`，除非缩小每个元素。

---


## 代码执行

### 基本用法

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  messages: [
    {
      role: "user",
      content:
        "Calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});
```

### 读取本地文件（ESM 注意）

`__dirname` 在 ES 模块中不存在。对于脚本相对路径，使用 `import.meta.url`：

```typescript
import { readFileSync } from "fs";
import { fileURLToPath } from "url";
import { dirname, join } from "path";

const __dirname = dirname(fileURLToPath(import.meta.url));
const pdfBytes = readFileSync(join(__dirname, "sample.pdf"));
```

或者，如果脚本从已知目录运行，使用 CWD 相对路径：`readFileSync("./sample.pdf")`。

### 上传文件进行分析

```typescript
import Anthropic, { toFile } from "@anthropic-ai/sdk";
import { createReadStream } from "fs";

const client = new Anthropic();

// 1. 上传文件
const uploaded = await client.beta.files.upload({
  file: await toFile(createReadStream("sales_data.csv"), undefined, {
    type: "text/csv",
  }),
  betas: ["files-api-2025-04-14"],
});

// 2. 传递给代码执行
// 代码执行是 GA；Files API 仍然是 beta（通过 RequestOptions 传递）
const response = await client.messages.create(
  {
    model: "{{OPUS_ID}}",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: [
          {
            type: "text",
            text: "Analyze this sales data. Show trends and create a visualization.",
          },
          { type: "container_upload", file_id: uploaded.id },
        ],
      },
    ],
    tools: [{ type: "code_execution_20260120", name: "code_execution" }],
  },
  { headers: { "anthropic-beta": "files-api-2025-04-14" } },
);
```

### 检索生成的文件

```typescript
import path from "path";
import fs from "fs";

const OUTPUT_DIR = "./claude_outputs";
await fs.promises.mkdir(OUTPUT_DIR, { recursive: true });

for (const block of response.content) {
  if (block.type === "bash_code_execution_tool_result") {
    const result = block.content;
    if (result.type === "bash_code_execution_result" && result.content) {
      for (const fileRef of result.content) {
        if (fileRef.type === "bash_code_execution_output") {
          const metadata = await client.beta.files.retrieveMetadata(
            fileRef.file_id,
          );
          const downloadResponse = await client.beta.files.download(fileRef.file_id);
          const fileBytes = Buffer.from(await downloadResponse.arrayBuffer());
          const safeName = path.basename(metadata.filename);
          if (!safeName || safeName === "." || safeName === "..") {
            console.warn(`Skipping invalid filename: ${metadata.filename}`);
            continue;
          }
          const outputPath = path.join(OUTPUT_DIR, safeName);
          await fs.promises.writeFile(outputPath, fileBytes);
          console.log(`Saved: ${outputPath}`);
        }
      }
    }
  }
}
```

### 容器重用

```typescript
// 第一次请求：设置环境
const response1 = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  messages: [
    {
      role: "user",
      content: "Install tabulate and create data.json with sample user data",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});

// 重用容器
// container 可为空 — 仅在使用服务端代码执行时设置
const containerId = response1.container!.id;

const response2 = await client.messages.create({
  container: containerId,
  model: "{{OPUS_ID}}",
  max_tokens: 4096,
  messages: [
    {
      role: "user",
      content: "Read data.json and display as a formatted table",
    },
  ],
  tools: [{ type: "code_execution_20260120", name: "code_execution" }],
});
```

---

## 内存工具

### 基本用法

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 2048,
  messages: [
    {
      role: "user",
      content: "Remember that my preferred language is TypeScript.",
    },
  ],
  tools: [{ type: "memory_20250818", name: "memory" }],
});
```

### SDK 内存辅助器

使用 `betaMemoryTool` 和 `MemoryToolHandlers` 实现：

```typescript
import {
  betaMemoryTool,
  type MemoryToolHandlers,
} from "@anthropic-ai/sdk/helpers/beta/memory";

const handlers: MemoryToolHandlers = {
  async view(command) { ... },
  async create(command) { ... },
  async str_replace(command) { ... },
  async insert(command) { ... },
  async delete(command) { ... },
  async rename(command) { ... },
};

const memory = betaMemoryTool(handlers);

const runner = client.beta.messages.toolRunner({
  model: "{{OPUS_ID}}",
  max_tokens: 2048,
  tools: [memory],
  messages: [{ role: "user", content: "Remember my preferences" }],
});

for await (const message of runner) {
  console.log(message);
}
```

有关完整的实现示例，请使用 WebFetch：

- `https://github.com/anthropics/anthropic-sdk-typescript/blob/main/examples/tools-helpers-memory.ts`

---

## 结构化输出

### JSON 输出（Zod — 推荐）

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { z } from "zod";
import { zodOutputFormat } from "@anthropic-ai/sdk/helpers/zod";

const ContactInfoSchema = z.object({
  name: z.string(),
  email: z.string(),
  plan: z.string(),
  interests: z.array(z.string()),
  demo_requested: z.boolean(),
});

const client = new Anthropic();

const response = await client.messages.parse({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content:
        "Extract: Jane Doe (jane@co.com) wants Enterprise, interested in API and SDKs, wants a demo.",
    },
  ],
  output_config: {
    format: zodOutputFormat(ContactInfoSchema),
  },
});

// 如果解析失败，parsed_output 为 null — 断言或保护
console.log(response.parsed_output!.name); // "Jane Doe"
```

### 严格工具使用

```typescript
const response = await client.messages.create({
  model: "{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: "Book a flight to Tokyo for 2 passengers on March 15",
    },
  ],
  tools: [
    {
      name: "book_flight",
      description: "Book a flight to a destination",
      strict: true,
      input_schema: {
        type: "object",
        properties: {
          destination: { type: "string" },
          date: { type: "string", format: "date" },
          passengers: {
            type: "integer",
            enum: [1, 2, 3, 4, 5, 6, 7, 8],
          },
        },
        required: ["destination", "date", "passengers"],
        additionalProperties: false,
      },
    },
  ],
});
```
