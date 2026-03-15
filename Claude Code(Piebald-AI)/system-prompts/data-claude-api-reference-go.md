<!--
name: 'Data: Claude API reference — Go'
description: Go SDK reference
ccVersion: 2.1.73
-->
# Claude API — Go

> **注意：** Go SDK 支持 Claude API 和 beta 工具使用，使用 \`BetaToolRunner\`。Agent SDK 尚不适用于 Go。

## 安装

\`\`\`bash
go get github.com/anthropics/anthropic-sdk-go
\`\`\`

## 客户端初始化

\`\`\`go
import (
    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/option"
)

// 默认（使用 ANTHROPIC_API_KEY 环境变量）
client := anthropic.NewClient()

// 显式 API 密钥
client := anthropic.NewClient(
    option.WithAPIKey("your-api-key"),
)
\`\`\`

---

## 基本消息请求

\`\`\`go
response, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
    Model:     anthropic.ModelClaudeOpus4_6,
    MaxTokens: 1024,
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("法国的首都是什么？")),
    },
})
if err != nil {
    log.Fatal(err)
}
for _, block := range response.Content {
    switch variant := block.AsAny().(type) {
    case anthropic.TextBlock:
        fmt.Println(variant.Text)
    }
}
\`\`\`

---

## 流式传输

\`\`\`go
stream := client.Messages.NewStreaming(context.Background(), anthropic.MessageNewParams{
    Model:     anthropic.ModelClaudeOpus4_6,
    MaxTokens: 1024,
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("写一首俳句")),
    },
})

for stream.Next() {
    event := stream.Current()
    switch eventVariant := event.AsAny().(type) {
    case anthropic.ContentBlockDeltaEvent:
        switch deltaVariant := eventVariant.Delta.AsAny().(type) {
        case anthropic.TextDelta:
            fmt.Print(deltaVariant.Text)
        }
    }
}
if err := stream.Err(); err != nil {
    log.Fatal(err)
}
\`\`\`

**累积最终消息**（流上没有 \`GetFinalMessage()\`）：

\`\`\`go
stream := client.Messages.NewStreaming(ctx, params)
message := anthropic.Message{}
for stream.Next() {
    message.Accumulate(stream.Current())
}
if err := stream.Err(); err != nil { log.Fatal(err) }
// message.Content 现在具有完整响应
\`\`\`


---

## 工具使用

### 工具运行器（Beta — 推荐）

**Beta：** Go SDK 通过 \`toolrunner\` 包为自动工具使用循环提供 \`BetaToolRunner\`。

\`\`\`go
import (
    "context"
    "fmt"
    "log"

    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/toolrunner"
)

// 使用 jsonschema 标签定义工具输入以自动生成模式
type GetWeatherInput struct {
    City string \`json:"city" jsonschema:"required,description=城市名称"\`
}

// 从结构标签创建具有自动模式生成的工具
weatherTool, err := toolrunner.NewBetaToolFromJSONSchema(
    "get_weather",
    "获取城市的当前天气",
    func(ctx context.Context, input GetWeatherInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
        return anthropic.BetaToolResultBlockParamContentUnion{
            OfText: &anthropic.BetaTextBlockParam{
                Text: fmt.Sprintf("%s 的天气晴朗，72°F", input.City),
            },
        }, nil
    },
)
if err != nil {
    log.Fatal(err)
}

// 创建自动处理对话循环的工具运行器
runner := client.Beta.Messages.NewToolRunner(
    []anthropic.BetaTool{weatherTool},
    anthropic.BetaToolRunnerParams{
        BetaMessageNewParams: anthropic.BetaMessageNewParams{
            Model:     anthropic.ModelClaudeOpus4_6,
            MaxTokens: 1024,
            Messages: []anthropic.BetaMessageParam{
                anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("巴黎的天气如何？")),
            },
        },
        MaxIterations: 5,
    },
)

// 运行直到 Claude 产生最终响应
message, err := runner.RunToCompletion(context.Background())
if err != nil {
    log.Fatal(err)
}

// RunToCompletion 返回 *BetaMessage；content 是 []BetaContentBlockUnion。
// 通过 AsAny() switch 缩小 — 注意 Beta 命名空间类型（BetaTextBlock，
// 不是 TextBlock）：
for _, block := range message.Content {
    switch block := block.AsAny().(type) {
    case anthropic.BetaTextBlock:
        fmt.Println(block.Text)
    }
}
\`\`\`

**Go 工具运行器的关键特性：**

- 通过 Go 结构中的 \`jsonschema\` 标记自动生成模式
- \`RunToCompletion()\` 用于简单的一次性使用
- \`All()\` 迭代器用于处理对话中的每条消息
- \`NextMessage()\` 用于逐步迭代
- 通过 \`NewToolRunnerStreaming()\` 和 \`AllStreaming()\` 的流式变体

### 手动循环

对于代理循环的精细控制，使用 \`ToolParam\` 定义工具，检查 \`StopReason\`，自己执行工具，并将 \`tool_result\` 块反馈。这是当您需要拦截、验证或记录工具调用时的模式。

源自 \`anthropic-sdk-go/examples/tools/main.go\`。

\`\`\`go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"

    "github.com/anthropics/anthropic-sdk-go"
)

func main() {
    client := anthropic.NewClient()

    // 1. 定义工具。ToolParam.InputSchema 使用映射，不需要结构标记。
    addTool := anthropic.ToolParam{
        Name:        "add",
        Description: anthropic.String("将两个整数相加"),
        InputSchema: anthropic.ToolInputSchemaParam{
            Properties: map[string]any{
                "a": map[string]any{"type": "integer"},
                "b": map[string]any{"type": "integer"},
            },
        },
    }
    // ToolParam 必须包装在 ToolUnionParam 中以用于 Tools 切片
    tools := []anthropic.ToolUnionParam{{OfTool: &addTool}}

    messages := []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("2 + 3 等于多少？")),
    }

    for {
        resp, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
            Model:     anthropic.ModelClaudeSonnet4_6,
            MaxTokens: 1024,
            Messages:  messages,
            Tools:     tools,
        })
        if err != nil {
            log.Fatal(err)
        }

        // 2. 在处理工具调用之前将助手响应追加到历史记录。
        //    resp.ToParam() 在一次调用中转换 Message → MessageParam。
        messages = append(messages, resp.ToParam())

        // 3. 遍历内容块。ContentBlockUnion 是一个扁平结构；
        //    使用 block.AsAny().(type) 切换实际变体。
        toolResults := []anthropic.ContentBlockParamUnion{}
        for _, block := range resp.Content {
            switch variant := block.AsAny().(type) {
            case anthropic.TextBlock:
                fmt.Println(variant.Text)
            case anthropic.ToolUseBlock:
                // 4. 解析工具输入。使用 variant.JSON.Input.Raw() 获取
                //    原始 JSON — block.Input 是 json.RawMessage，不是解析后的值。
                var in struct {
                    A int \`json:"a"\`
                    B int \`json:"b"\`
                }
                if err := json.Unmarshal([]byte(variant.JSON.Input.Raw()), &in); err != nil {
                    log.Fatal(err)
                }
                result := fmt.Sprintf("%d", in.A+in.B)
                // 5. NewToolResultBlock(toolUseID, content, isError) 为您
                //    构建 ContentBlockParamUnion。block.ID 是 tool_use_id。
                toolResults = append(toolResults,
                    anthropic.NewToolResultBlock(block.ID, result, false))
            }
        }

        // 6. 当 Claude 停止请求工具时退出
        if resp.StopReason != anthropic.StopReasonToolUse {
            break
        }

        // 7. 工具结果进入用户消息（可变参数：所有结果在一轮中）
        messages = append(messages, anthropic.NewUserMessage(toolResults...))
    }
}
\`\`\`

**关键 API 表面：**

| 符号 | 用途 |
|---|---|
| \`resp.ToParam()\` | 转换 \`Message\` 响应 → 用于历史记录的 \`MessageParam\` |
| \`block.AsAny().(type)\` | \`ContentBlockUnion\` 变体的类型切换 |
| \`variant.JSON.Input.Raw()\` | 工具输入的原始 JSON 字符串（用于 \`json.Unmarshal\`） |
| \`anthropic.NewToolResultBlock(id, content, isError)\` | 构建 \`tool_result\` 块 |
| \`anthropic.NewUserMessage(blocks...)\` | 将工具结果包装为用户轮次 |
| \`anthropic.StopReasonToolUse\` | \`StopReason\` 常量以检查循环终止 |
| \`anthropic.ToolUnionParam{OfTool: &t}\` | 将 \`ToolParam\` 包装在 \`Tools:\` 的联合中 |

---

## 思考

通过在 \`MessageNewParams\` 中设置 \`Thinking\` 来启用 Claude 的内部推理。响应将在最终的 \`TextBlock\` 之前包含 \`ThinkingBlock\` 内容。

**自适应思考是 Claude 4.6+ 模型的推荐模式。** Claude 动态决定何时以及多少思考。与 \`effort\` 参数结合以实现成本质量控制。

源自 \`anthropic-sdk-go/message.go\`（\`ThinkingConfigParamUnion\`、\`NewThinkingConfigAdaptiveParam\`）。

\`\`\`go
// 没有 ThinkingConfigParamOfAdaptive 辅助方法 — 直接构造联合
// 结构文字并采用变体的地址。
adaptive := anthropic.NewThinkingConfigAdaptiveParam()
params := anthropic.MessageNewParams{
    Model:     anthropic.ModelClaudeSonnet4_6,
    MaxTokens: 16000,
    Thinking:  anthropic.ThinkingConfigParamUnion{OfAdaptive: &adaptive},
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("草莓中有多少个 r？")),
    },
}

resp, err := client.Messages.New(context.Background(), params)
if err != nil {
    log.Fatal(err)
}

// ThinkingBlock 在 content 中的 TextBlock 之前
for _, block := range resp.Content {
    switch b := block.AsAny().(type) {
    case anthropic.ThinkingBlock:
        fmt.Println("[thinking]", b.Thinking)
    case anthropic.TextBlock:
        fmt.Println(b.Text)
    }
}
\`\`\`

> **已弃用：** \`ThinkingConfigParamOfEnabled(budgetTokens)\`（固定预算的扩展思考）在 Claude 4.6 上仍然有效，但已弃用。使用上面的自适应思考。

要禁用：\`anthropic.ThinkingConfigParamUnion{OfDisabled: &anthropic.ThinkingConfigDisabledParam{}}\`。

---

## 服务器端工具

带有 \`Param\` 后缀的版本后缀结构名称。\`Name\`/\`Type\` 是 \`constant.*\` 类型 — 零值正确编组，因此 \`{}\` 有效。使用匹配的 \`Of*\` 字段包装在 \`ToolUnionParam\` 中。

\`\`\`go
Tools: []anthropic.ToolUnionParam{
    {OfWebSearchTool20260209: &anthropic.WebSearchTool20260209Param{}},
    {OfBashTool20250124: &anthropic.ToolBash20250124Param{}},
    {OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{}},
    {OfCodeExecutionTool20260120: &anthropic.CodeExecutionTool20260120Param{}},
},
\`\`\`

也可用：\`WebFetchTool20260209Param\`、\`MemoryTool20250818Param\`、\`ToolSearchToolBm25_20251119Param\`、\`ToolSearchToolRegex20251119Param\`。

---

## PDF / 文档输入

\`NewDocumentBlock\` 通用辅助方法接受任何源类型。\`MediaType\`/\`Type\` 自动设置。

\`\`\`go
b64 := base64.StdEncoding.EncodeToString(pdfBytes)

msg := anthropic.NewUserMessage(
    anthropic.NewDocumentBlock(anthropic.Base64PDFSourceParam{Data: b64}),
    anthropic.NewTextBlock("总结此文档"),
)
\`\`\`

其他源：\`URLPDFSourceParam{URL: "https://..."}\`、\`PlainTextSourceParam{Data: "..."}`。

---

## Files API（Beta）

在 \`client.Beta.Files\` 下。方法是 **\`Upload\`**（不是 \`New\`/\`Create\`），参数结构是 \`BetaFileUploadParams\`。\`File\` 字段接受 \`io.Reader\`；使用 \`anthropic.File()\` 附加文件名和内容类型以进行多部分编码。

\`\`\`go
f, _ := os.Open("./upload_me.txt")
defer f.Close()

meta, err := client.Beta.Files.Upload(ctx, anthropic.BetaFileUploadParams{
    File:  anthropic.File(f, "upload_me.txt", "text/plain"),
    Betas: []anthropic.AnthropicBeta{anthropic.AnthropicBetaFilesAPI2025_04_14},
})
// meta.ID 是要在后续消息请求中引用的 file_id
\`\`\`

其他 \`Beta.Files\` 方法：\`List\`、\`Delete\`、\`Download\`、\`GetMetadata\`。

---

## 上下文编辑/压缩（Beta）

使用 \`Beta.Messages.New\` 和 \`BetaMessageNewParams\` 上的 \`ContextManagement\`。没有 \`NewBetaAssistantMessage\` — 使用 \`.ToParam()\` 进行往返。

\`\`\`go
params := anthropic.BetaMessageNewParams{
    Model:     anthropic.ModelClaudeOpus4_6,  // 也支持：ModelClaudeSonnet4_6
    MaxTokens: 1024,
    Betas:     []anthropic.AnthropicBeta{"compact-2026-01-12"},
    ContextManagement: anthropic.BetaContextManagementConfigParam{
        Edits: []anthropic.BetaContextManagementConfigEditUnionParam{
            {OfCompact20260112: &anthropic.BetaCompact20260112EditParam{}},
        },
    },
    Messages: []anthropic.BetaMessageParam{ /* ... */ },
}

resp, err := client.Beta.Messages.New(ctx, params)
if err != nil {
    log.Fatal(err)
}

// 往返：通过 .ToParam() 将响应追加到历史记录
params.Messages = append(params.Messages, resp.ToParam())

// 从响应中读取压缩块
for _, block := range resp.Content {
    if c, ok := block.AsAny().(anthropic.BetaCompactionBlock); ok {
        fmt.Println("压缩摘要：", c.Content)
    }
}
\`\`\`

其他编辑类型：\`BetaClearToolUses20250919EditParam\`、\`BetaClearThinking20251015EditParam\`。
