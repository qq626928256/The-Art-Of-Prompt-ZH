<!--
name: 'Data: Claude API reference — Ruby'
description: Ruby SDK reference including installation, client initialization, basic requests, streaming, and beta tool runner
ccVersion: 2.1.71
-->
# Claude API — Ruby

> **注意：** Ruby SDK 支持 Claude API。通过 `client.beta.messages.tool_runner()` 提供了 beta 工具运行器。Agent SDK 尚不适用于 Ruby。

## 安装

```bash
gem install anthropic
```

## 客户端初始化

```ruby
require "anthropic"

# 默认（使用 ANTHROPIC_API_KEY 环境变量）
client = Anthropic::Client.new

# 显式 API 密钥
client = Anthropic::Client.new(api_key: "your-api-key")
```

---

## 基本消息请求

```ruby
message = client.messages.create(
  model: :"{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "What is the capital of France?" }
  ]
)
# content 是多态块对象数组（TextBlock、ThinkingBlock、
# ToolUseBlock，...）。.type 是一个 Symbol — 与 :text 比较，而不是 "text"。
# 在非 TextBlock 条目上 .text 会引发 NoMethodError。
message.content.each do |block|
  puts block.text if block.type == :text
end
```

---

## 流式传输

```ruby
stream = client.messages.stream(
  model: :"{{OPUS_ID}}",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
```

---

## 工具使用

Ruby SDK 通过原始 JSON schema 定义支持工具使用，并且还提供了用于自动工具执行的 beta 工具运行器。

### 工具运行器（Beta）

```ruby
class GetWeatherInput < Anthropic::BaseModel
  required :location, String, doc: "City and state, e.g. San Francisco, CA"
end

class GetWeather < Anthropic::BaseTool
  doc "Get the current weather for a location"

  input_schema GetWeatherInput

  def call(input)
    "The weather in #{input.location} is sunny and 72°F."
  end
end

client.beta.messages.tool_runner(
  model: :"{{OPUS_ID}}",
  max_tokens: 1024,
  tools: [GetWeather.new],
  messages: [{ role: "user", content: "What's the weather in San Francisco?" }]
).each_message do |message|
  puts message.content
end
```

### 手动循环

参见[共享工具使用概念](../shared/tool-use-concepts.md)了解工具定义格式和 agentic 循环模式。
