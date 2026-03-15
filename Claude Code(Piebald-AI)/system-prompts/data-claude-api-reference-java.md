<!--
name: 'Data: Claude API reference — Java'
description: Java SDK reference including installation, client initialization, basic requests, streaming, and beta tool use
ccVersion: 2.1.73
-->
# Claude API — Java

> **注意：** Java SDK 支持 Claude API 和带注解类的 beta 工具使用。Agent SDK 尚不适用于 Java。

## 安装

Maven:

```xml
<dependency>
    <groupId>com.anthropic</groupId>
    <artifactId>anthropic-java</artifactId>
    <version>2.15.0</version>
</dependency>
```

Gradle:

```groovy
implementation("com.anthropic:anthropic-java:2.15.0")
```

## 客户端初始化

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

// 默认（从环境变量读取 ANTHROPIC_API_KEY）
AnthropicClient client = AnthropicOkHttpClient.fromEnv();

// 显式 API 密钥
AnthropicClient client = AnthropicOkHttpClient.builder()
    .apiKey("your-api-key")
    .build();
```

---

## 基本消息请求

```java
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.Model;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(1024L)
    .addUserMessage("What is the capital of France?")
    .build();

Message response = client.messages().create(params);
response.content().stream()
    .flatMap(block -> block.text().stream())
    .forEach(textBlock -> System.out.println(textBlock.text()));
```

---

## 流式传输

```java
import com.anthropic.core.http.StreamResponse;
import com.anthropic.models.messages.RawMessageStreamEvent;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(1024L)
    .addUserMessage("Write a haiku")
    .build();

try (StreamResponse<RawMessageStreamEvent> streamResponse = client.messages().createStreaming(params)) {
    streamResponse.stream()
        .flatMap(event -> event.contentBlockDelta().stream())
        .flatMap(deltaEvent -> deltaEvent.delta().text().stream())
        .forEach(textDelta -> System.out.print(textDelta.text()));
}
```

---

## 思考

**自适应思考是 Claude 4.6+ 模型的推荐模式。** Claude 会动态决定何时思考以及思考多少。构建器有直接的 `.thinking(ThinkingConfigAdaptive)` 重载 — 无需手动包装联合类型。

```java
import com.anthropic.models.messages.ContentBlock;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;
import com.anthropic.models.messages.ThinkingConfigAdaptive;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_SONNET_4_6)
    .maxTokens(16000L)
    .thinking(ThinkingConfigAdaptive.builder().build())
    .addUserMessage("Solve this step by step: 27 * 453")
    .build();

for (ContentBlock block : client.messages().create(params).content()) {
    block.thinking().ifPresent(t -> System.out.println("[thinking] " + t.thinking()));
    block.text().ifPresent(t -> System.out.println(t.text()));
}
```

> **已弃用：** `ThinkingConfigEnabled.builder().budgetTokens(N)`（以及 `.enabledThinking(N)` 快捷方式）在 Claude 4.6 上仍然可用，但已弃用。请使用上面的自适应思考。

`ContentBlock` 缩小：`.thinking()` / `.text()` 返回 `Optional<T>` — 使用 `.ifPresent(...)` 或 `.stream().flatMap(...)`。替代方案：`isThinking()` / `asThinking()` 布尔值+解包对（在错误变体上抛出异常）。

---

## 工具使用（Beta）

Java SDK 支持带注解类的 beta 工具使用。工具类实现 `Supplier<String>` 以通过 `BetaToolRunner` 自动执行。

### 工具运行器（自动循环）

```java
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.helpers.BetaToolRunner;
import com.fasterxml.jackson.annotation.JsonClassDescription;
import com.fasterxml.jackson.annotation.JsonPropertyDescription;
import java.util.function.Supplier;

@JsonClassDescription("Get the weather in a given location")
static class GetWeather implements Supplier<String> {
    @JsonPropertyDescription("The city and state, e.g. San Francisco, CA")
    public String location;

    @Override
    public String get() {
        return "The weather in " + location + " is sunny and 72°F";
    }
}

BetaToolRunner toolRunner = client.beta().messages().toolRunner(
    MessageCreateParams.builder()
        .model("{{OPUS_ID}}")
        .maxTokens(1024L)
        .putAdditionalHeader("anthropic-beta", "structured-outputs-2025-11-13")
        .addTool(GetWeather.class)
        .addUserMessage("What's the weather in San Francisco?")
        .build());

for (BetaMessage message : toolRunner) {
    System.out.println(message);
}
```

### 非 Beta 工具声明（手动 JSON schema）

`Tool.InputSchema.Properties` 是一个自由格式的 `Map<String, JsonValue>` 包装器 — 通过 `putAdditionalProperty` 构建属性 schema。`type: "object"` 是默认值。构建器有直接的 `.addTool(Tool)` 重载，会自动包装在 `ToolUnion` 中。

```java
import com.anthropic.core.JsonValue;
import com.anthropic.models.messages.Tool;

Tool tool = Tool.builder()
    .name("get_weather")
    .description("Get the current weather in a given location")
    .inputSchema(Tool.InputSchema.builder()
        .properties(Tool.InputSchema.Properties.builder()
            .putAdditionalProperty("location", JsonValue.from(Map.of("type", "string")))
            .build())
        .required(List.of("location"))
        .build())
    .build();

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_SONNET_4_6)
    .maxTokens(1024L)
    .addTool(tool)
    .addUserMessage("Weather in Paris?")
    .build();
```

对于手动工具循环，处理响应中的 `tool_use` 块，将 `tool_result` 发回，循环直到 `stop_reason` 为 `"end_turn"`。参见[共享工具使用概念](../shared/tool-use-concepts.md)。

### 使用内容块构建 `MessageParam`（工具结果往返）

`MessageParam.Content` 是一个内部联合类（string | list）。使用构建器的 `.contentOfBlockParams(List<ContentBlockParam>)` 别名 — 没有单独的 `MessageParamContent` 类带有静态 `ofBlockParams`：

```java
import com.anthropic.models.messages.MessageParam;
import com.anthropic.models.messages.ContentBlockParam;
import com.anthropic.models.messages.ToolResultBlockParam;

List<ContentBlockParam> results = List.of(
    ContentBlockParam.ofToolResult(ToolResultBlockParam.builder()
        .toolUseId(toolUseBlock.id())
        .content(yourResultString)
        .build())
);

MessageParam toolResultMsg = MessageParam.builder()
    .role(MessageParam.Role.USER)
    .contentOfBlockParams(results)   // Content.ofBlockParams(...) 的构建器别名
    .build();
```

---

## Effort 参数

Effort 嵌套在 `OutputConfig` 内 — `MessageCreateParams.Builder` 上没有直接的 `.effort()`。

```java
import com.anthropic.models.messages.OutputConfig;

.outputConfig(OutputConfig.builder()
    .effort(OutputConfig.Effort.HIGH)  // 或 LOW、MEDIUM、MAX
    .build())
```

结合 `Thinking = ThinkingConfigAdaptive` 以实现成本质量控制。

---

## 提示缓存

系统消息作为带有 `CacheControlEphemeral` 的 `TextBlockParam` 列表。使用 `.systemOfTextBlockParams(...)` — 简单的 `.system(String)` 重载无法携带缓存控制。

```java
import com.anthropic.models.messages.TextBlockParam;
import com.anthropic.models.messages.CacheControlEphemeral;

.systemOfTextBlockParams(List.of(
    TextBlockParam.builder()
        .text(longSystemPrompt)
        .cacheControl(CacheControlEphemeral.builder()
            .ttl(CacheControlEphemeral.Ttl.TTL_1H)  // 可选；也有 TTL_5M
            .build())
        .build()))
```

在 `MessageCreateParams.Builder` 和 `Tool.builder()` 上也有顶级的 `.cacheControl(CacheControlEphemeral)`。

---

## Token 计数

```java
import com.anthropic.models.messages.MessageCountTokensParams;

long tokens = client.messages().countTokens(
    MessageCountTokensParams.builder()
        .model(Model.CLAUDE_SONNET_4_6)
        .addUserMessage("Hello")
        .build()
).inputTokens();
```

---

## 结构化输出

基于类的重载会自动从您的 POJO 派生 JSON schema，并为您提供类型化的 `.text()` 返回值 — 无需手动 schema，无需手动解析。

```java
import com.anthropic.models.messages.StructuredMessageCreateParams;

record Book(String title, String author) {}
record BookList(List<Book> books) {}

StructuredMessageCreateParams<BookList> params = MessageCreateParams.builder()
    .model(Model.CLAUDE_SONNET_4_6)
    .maxTokens(2048L)
    .outputConfig(BookList.class)  // 返回类型化的构建器
    .addUserMessage("List 3 classic novels")
    .build();

client.messages().create(params).content().stream()
    .flatMap(cb -> cb.text().stream())
    .forEach(typed -> {
        // typed.text() 返回 BookList，而不是 String
        for (Book b : typed.text().books()) System.out.println(b.title());
    });
```

支持 Jackson 注解：`@JsonPropertyDescription`、`@JsonIgnore`、`@ArraySchema(minItems=...)`。手动 schema 路径：`OutputConfig.builder().format(JsonOutputFormat.builder().schema(...).build())`。

---

## PDF / 文档输入

`DocumentBlockParam` 构建器有 source 快捷方式。包装在 `ContentBlockParam.ofDocument()` 中并通过 `.addUserMessageOfBlockParams()` 传递。

```java
import com.anthropic.models.messages.DocumentBlockParam;
import com.anthropic.models.messages.ContentBlockParam;
import com.anthropic.models.messages.TextBlockParam;

DocumentBlockParam doc = DocumentBlockParam.builder()
    .base64Source(base64String)  // 或 .urlSource("https://...") 或 .textSource("...")
    .title("My Document")        // 可选
    .build();

.addUserMessageOfBlockParams(List.of(
    ContentBlockParam.ofDocument(doc),
    ContentBlockParam.ofText(TextBlockParam.builder().text("Summarize this").build())))
```

---

## 服务端工具

带版本后缀的类型；`name`/`type` 由构建器自动设置。每种类型都有直接的 `.addTool()` 重载 — 无需手动 `ToolUnion` 包装。

```java
import com.anthropic.models.messages.WebSearchTool20260209;
import com.anthropic.models.messages.ToolBash20250124;
import com.anthropic.models.messages.ToolTextEditor20250728;
import com.anthropic.models.messages.CodeExecutionTool20260120;

.addTool(WebSearchTool20260209.builder()
    .maxUses(5L)                              // 可选
    .allowedDomains(List.of("example.com"))   // 可选
    .build())
.addTool(ToolBash20250124.builder().build())
.addTool(ToolTextEditor20250728.builder().build())
.addTool(CodeExecutionTool20260120.builder().build())
```

也可用：`WebFetchTool20260209`、`MemoryTool20250818`、`ToolSearchToolBm25_20251119`。

### Beta 命名空间（MCP、压缩）

对于仅 beta 功能，使用 `com.anthropic.models.beta.messages.*` — 类名有 `Beta` 前缀并且位于 beta 包中。beta `MessageCreateParams.Builder` 有直接的 `.addTool(BetaToolBash20250124)` 重载和 `.addMcpServer()`：

```java
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaToolBash20250124;
import com.anthropic.models.beta.messages.BetaCodeExecutionTool20260120;
import com.anthropic.models.beta.messages.BetaRequestMcpServerUrlDefinition;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(1024L)
    .addBeta("mcp-client-2025-11-20")
    .addTool(BetaToolBash20250124.builder().build())
    .addTool(BetaCodeExecutionTool20260120.builder().build())
    .addMcpServer(BetaRequestMcpServerUrlDefinition.builder()
        .name("my-server")
        .url("https://example.com/mcp")
        .build())
    .addUserMessage("...")
    .build();

client.beta().messages().create(params);
```

`BetaTool*` 类型与非 beta `Tool*` 不可互换 — 每个请求选择一个命名空间。

**在响应中读取服务端工具块：** `ServerToolUseBlock` 有 `.id()`、`.name()`（枚举）和 `._input()` 返回原始 `JsonValue` — 没有类型化的 `.input()`。对于代码执行结果，解包两个级别：

```java
for (ContentBlock block : response.content()) {
    block.serverToolUse().ifPresent(stu -> {
        System.out.println("tool: " + stu.name() + " input: " + stu._input());
    });
    block.codeExecutionToolResult().ifPresent(r -> {
        r.content().resultBlock().ifPresent(result -> {
            System.out.println("stdout: " + result.stdout());
            System.out.println("stderr: " + result.stderr());
            System.out.println("exit: " + result.returnCode());
        });
    });
}
```

---

## Files API（Beta）

在 `client.beta().files()` 下。消息中的文件引用需要 beta 消息类型（非 beta `DocumentBlockParam.Source` 没有文件 ID 变体）。

```java
import com.anthropic.models.beta.files.FileUploadParams;
import com.anthropic.models.beta.files.FileMetadata;
import com.anthropic.models.beta.messages.BetaRequestDocumentBlock;
import java.nio.file.Paths;

FileMetadata meta = client.beta().files().upload(
    FileUploadParams.builder()
        .file(Paths.get("/path/to/doc.pdf"))  // 或 .file(InputStream) 或 .file(byte[])
        .build());

// 在 beta 消息中引用：
BetaRequestDocumentBlock doc = BetaRequestDocumentBlock.builder()
    .fileSource(meta.id())
    .build();
```

其他方法：`.list()`、`.delete(String fileId)`、`.download(String fileId)`、`.retrieveMetadata(String fileId)`。
