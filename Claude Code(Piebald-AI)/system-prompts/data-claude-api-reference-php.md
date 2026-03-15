<!--
name: 'Data: Claude API reference — PHP'
description: PHP SDK reference
ccVersion: 2.1.73
-->
# Claude API — PHP

> **注意：** PHP SDK 是 Anthropic 的官方 PHP SDK。工具运行器和 Agent SDK 不可用。支持 Bedrock、Vertex AI 和 Foundry 客户端。

## 安装

```bash
composer require "anthropic-ai/sdk"
```

## 客户端初始化

```php
use Anthropic\Client;

// 使用环境变量中的 API 密钥
$client = new Client(apiKey: getenv("ANTHROPIC_API_KEY"));
```

### Amazon Bedrock

```php
use Anthropic\Bedrock;

// 构造函数是私有的 — 使用静态工厂。从环境变量读取 AWS 凭证。
$client = Bedrock\Client::fromEnvironment(region: 'us-east-1');
```

### Google Vertex AI

```php
use Anthropic\Vertex;

// 构造函数是私有的。参数是 `location`，不是 `region`。
$client = Vertex\Client::fromEnvironment(
    location: 'us-east5',
    projectId: 'my-project-id',
);
```

### Anthropic Foundry

```php
use Anthropic\Foundry;

// 构造函数是私有的。baseUrl 或 resource 是必需的。
$client = Foundry\Client::withCredentials(
    authToken: getenv('ANTHROPIC_FOUNDRY_AUTH_TOKEN'),
    baseUrl: 'https://<resource>.services.ai.azure.com/anthropic',
);
```

---

## 基本消息请求

```php
$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 1024,
    messages: [
        ['role' => 'user', 'content' => 'What is the capital of France?'],
    ],
);

// content 是多态块数组（TextBlock、ToolUseBlock、
// ThinkingBlock）。如果不检查块类型就访问 ->text
// 会在第一个块不是 TextBlock 时抛出异常（例如，当启用了
// 扩展思考且 ThinkingBlock 先出现时）。始终使用保护：
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```

如果只需要第一个文本块：

```php
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
        break;
    }
}
```

---

## 流式传输

> **需要 SDK v0.5.0+。** v0.4.0 及更早版本使用单个 `$params` 数组；使用命名参数调用会抛出 `Unknown named parameter $model`。升级：`composer require "anthropic-ai/sdk:^0.6"`

```php
use Anthropic\Messages\RawContentBlockDeltaEvent;
use Anthropic\Messages\TextDelta;

$stream = $client->messages->createStream(
    model: '{{OPUS_ID}}',
    maxTokens: 1024,
    messages: [
        ['role' => 'user', 'content' => 'Write a haiku'],
    ],
);

foreach ($stream as $event) {
    if ($event instanceof RawContentBlockDeltaEvent && $event->delta instanceof TextDelta) {
        echo $event->delta->text;
    }
}
```

---

## 工具使用（手动循环）

工具作为数组传递。**SDK 使用 camelCase 键**（`inputSchema`、`toolUseID`、`stopReason`）并自动映射到 API 的 snake_case — 自 v0.5.0 起。参见[共享工具使用概念](../shared/tool-use-concepts.md)了解循环模式。

```php
use Anthropic\Messages\ToolUseBlock;

$tools = [
    [
        'name' => 'get_weather',
        'description' => 'Get the current weather in a given location',
        'inputSchema' => [  // camelCase，不是 input_schema
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City and state'],
            ],
            'required' => ['location'],
        ],
    ],
];

$messages = [['role' => 'user', 'content' => 'What is the weather in SF?']];

$response = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 1024,
    tools: $tools,
    messages: $messages,
);

while ($response->stopReason === 'tool_use') {  // camelCase 属性
    $toolResults = [];
    foreach ($response->content as $block) {
        if ($block instanceof ToolUseBlock) {
            // $block->name  : string               — 用于分发的工具名称
            // $block->input : array<string,mixed>  — 解析的 JSON 输入
            // $block->id    : string               — 作为 toolUseID 传回
            $result = executeYourTool($block->name, $block->input);
            $toolResults[] = [
                'type' => 'tool_result',
                'toolUseID' => $block->id,  // camelCase，不是 tool_use_id
                'content' => $result,
            ];
        }
    }

    // 附加助手轮次 + 带有工具结果的用户轮次
    $messages[] = ['role' => 'assistant', 'content' => $response->content];
    $messages[] = ['role' => 'user', 'content' => $toolResults];

    $response = $client->messages->create(
        model: '{{OPUS_ID}}',
        maxTokens: 1024,
        tools: $tools,
        messages: $messages,
    );
}

// 最终文本响应
foreach ($response->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```

`$block->type === 'tool_use'` 也可以；`instanceof ToolUseBlock` 为 PHPStan 缩小类型。


---

## 扩展思考

**自适应思考是 Claude 4.6+ 模型的推荐模式。** Claude 会动态决定何时思考以及思考多少。

```php
use Anthropic\Messages\ThinkingBlock;

$message = $client->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 16000,
    thinking: ['type' => 'adaptive'],
    messages: [
        ['role' => 'user', 'content' => 'Solve: 27 * 453'],
    ],
);

// ThinkingBlock 在 content 中位于 TextBlock 之前
foreach ($message->content as $block) {
    if ($block instanceof ThinkingBlock) {
        echo "Thinking:\n{$block->thinking}\n\n";
        // $block->signature 是一个不透明的字符串 — 如果在多轮对话中
        // 传回思考块，请逐字保留
    } elseif ($block->type === 'text') {
        echo "Answer: {$block->text}\n";
    }
}
```

> **已弃用：** `['type' => 'enabled', 'budgetTokens' => N]`（固定预算扩展思考）在 Claude 4.6 上仍然可用，但已弃用。请使用上面的自适应思考。

`$block->type === 'thinking'` 也可以用于检查；`instanceof` 为 PHPStan 缩小类型。

---

## Beta 功能和服务端工具

**`betas:` 不是 `$client->messages->create()` 上的参数** — 它只存在于 beta 命名空间上。将其用于需要显式选择加入 header 的功能：

```php
use Anthropic\Beta\Messages\BetaRequestMCPServerURLDefinition;

$response = $client->beta->messages->create(
    model: '{{OPUS_ID}}',
    maxTokens: 1024,
    mcpServers: [
        BetaRequestMCPServerURLDefinition::with(
            name: 'my-server',
            url: 'https://example.com/mcp',
        ),
    ],
    betas: ['mcp-client-2025-11-20'],  // 仅在 ->beta->messages 上有效
    messages: [['role' => 'user', 'content' => 'Use the MCP tools']],
);
```

**服务端工具**（bash、web_search、text_editor、code_execution）是 GA，在两个路径上都可用 — 非 beta 使用 `Anthropic\Messages\ToolBash20250124` / `WebSearchTool20260209` / `ToolTextEditor20250728` / `CodeExecutionTool20260120`，beta 使用 `Anthropic\Beta\Messages\BetaToolBash20250124` / `BetaWebSearchTool20260209` / `BetaToolTextEditor20250728` / `BetaCodeExecutionTool20260120`。这些不需要 `betas:` header。
