<!--
name: 'Data: Tool use reference — Python'
description: Python tool use reference including tool runner, manual agentic loop, code execution, and structured outputs
ccVersion: 2.1.73
-->
# 工具使用 — Python

有关概念概述（工具定义、工具选择、技巧），请参阅 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## 工具运行器（推荐）

**Beta：** Python SDK 中的工具运行器处于 beta 阶段。

使用 `@beta_tool` 装饰器将工具定义为类型化函数，然后将它们传递给 `client.beta.messages.tool_runner()`：

```python
import anthropic
from anthropic import beta_tool

client = anthropic.Anthropic()

@beta_tool
def get_weather(location: str, unit: str = "celsius") -> str:
    """Get current weather for a location.

    Args:
        location: City and state, e.g., San Francisco, CA.
        unit: Temperature unit, either "celsius" or "fahrenheit".
    """
    # 您的实现在这里
    return f"72°F and sunny in {location}"

# 工具运行器自动处理 agentic 循环
runner = client.beta.messages.tool_runner(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    tools=[get_weather],
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
)

# 每次迭代产生一个 BetaMessage；当 Claude 完成时迭代停止
for message in runner:
    print(message)
```

对于异步使用，将 `@beta_async_tool` 与 `async def` 函数一起使用。

**工具运行器的主要好处：**

- 无需手动循环 — SDK 处理调用工具和反馈结果
- 通过装饰器实现类型安全的工具输入
- 工具 schema 从函数签名自动生成
- 当 Claude 没有更多工具调用时迭代自动停止

---

## MCP 工具转换辅助器

**Beta。** 将 [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) 工具、提示和资源转换为 Anthropic API 类型，以便与工具运行器一起使用。需要 `pip install anthropic[mcp]`（Python 3.10+）。

> **注意：** Claude API 还支持 `mcp_servers` 参数，允许 Claude 直接连接到远程 MCP 服务器。当您需要本地 MCP 服务器、提示、资源或对 MCP 连接进行更多控制时，请使用这些辅助器。

### 使用工具运行器的 MCP 工具

```python
from anthropic import AsyncAnthropic
from anthropic.lib.tools.mcp import async_mcp_tool
from mcp import ClientSession
from mcp.client.stdio import stdio_client, StdioServerParameters

client = AsyncAnthropic()

async with stdio_client(StdioServerParameters(command="mcp-server")) as (read, write):
    async with ClientSession(read, write) as mcp_client:
        await mcp_client.initialize()

        tools_result = await mcp_client.list_tools()
        # tool_runner 是同步的 — 返回运行器，而不是协程
        runner = client.beta.messages.tool_runner(
            model="{{OPUS_ID}}",
            max_tokens=1024,
            messages=[{"role": "user", "content": "Use the available tools"}],
            tools=[async_mcp_tool(t, mcp_client) for t in tools_result.tools],
        )
        async for message in runner:
            print(message)
```

对于同步使用，使用 `mcp_tool` 而不是 `async_mcp_tool`。

### MCP 提示

```python
from anthropic.lib.tools.mcp import mcp_message

prompt = await mcp_client.get_prompt(name="my-prompt")
response = await client.beta.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[mcp_message(m) for m in prompt.messages],
)
```

### MCP 资源作为内容

```python
from anthropic.lib.tools.mcp import mcp_resource_to_content

resource = await mcp_client.read_resource(uri="file:///path/to/doc.txt")
response = await client.beta.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            mcp_resource_to_content(resource),
            {"type": "text", "text": "Summarize this document"},
        ],
    }],
)
```

### 上传 MCP 资源作为文件

```python
from anthropic.lib.tools.mcp import mcp_resource_to_file

resource = await mcp_client.read_resource(uri="file:///path/to/data.json")
uploaded = await client.beta.files.upload(file=mcp_resource_to_file(resource))
```

如果无法转换 MCP 值（例如，不支持的 content 类型如音频、不支持的 MIME 类型），转换函数会引发 `UnsupportedMCPValueError`。

---

## 手动 Agentic 循环

当您需要对循环进行细粒度控制时使用此方法（例如，自定义日志记录、条件工具执行、人工批准循环）：

```python
import anthropic

client = anthropic.Anthropic()
tools = [...]  # 您的工具定义
messages = [{"role": "user", "content": user_input}]

# Agentic 循环：继续直到 Claude 停止调用工具
while True:
    response = client.messages.create(
        model="{{OPUS_ID}}",
        max_tokens=4096,
        tools=tools,
        messages=messages
    )

    # 如果 Claude 完成（没有更多工具调用），中断
    if response.stop_reason == "end_turn":
        break

    # 服务端工具达到迭代限制；重新发送以继续
    if response.stop_reason == "pause_turn":
        messages = [
            {"role": "user", "content": user_input},
            {"role": "assistant", "content": response.content},
        ]
        continue

    # 从响应中提取工具使用块
    tool_use_blocks = [b for b in response.content if b.type == "tool_use"]

    # 附加助手的响应（包括 tool_use 块）
    messages.append({"role": "assistant", "content": response.content})

    # 执行每个工具并收集结果
    tool_results = []
    for tool in tool_use_blocks:
        result = execute_tool(tool.name, tool.input)  # 您的实现
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": tool.id,  # 必须匹配 tool_use 块的 id
            "content": result
        })

    # 将工具结果作为用户消息附加
    messages.append({"role": "user", "content": tool_results})

# 最终响应文本
final_text = next(b.text for b in response.content if b.type == "text")
```

---

## 处理工具结果

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Paris?"}]
)

for block in response.content:
    if block.type == "tool_use":
        tool_name = block.name
        tool_input = block.input
        tool_use_id = block.id

        result = execute_tool(tool_name, tool_input)

        followup = client.messages.create(
            model="{{OPUS_ID}}",
            max_tokens=1024,
            tools=tools,
            messages=[
                {"role": "user", "content": "What's the weather in Paris?"},
                {"role": "assistant", "content": response.content},
                {
                    "role": "user",
                    "content": [{
                        "type": "tool_result",
                        "tool_use_id": tool_use_id,
                        "content": result
                    }]
                }
            ]
        )
```

---

## 多个工具调用

```python
tool_results = []

for block in response.content:
    if block.type == "tool_use":
        result = execute_tool(block.name, block.input)
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": result
        })

# 一次性发回所有结果
if tool_results:
    followup = client.messages.create(
        model="{{OPUS_ID}}",
        max_tokens=1024,
        tools=tools,
        messages=[
            *previous_messages,
            {"role": "assistant", "content": response.content},
            {"role": "user", "content": tool_results}
        ]
    )
```

---

## 工具结果中的错误处理

```python
tool_result = {
    "type": "tool_result",
    "tool_use_id": tool_use_id,
    "content": "Error: Location 'xyz' not found. Please provide a valid city name.",
    "is_error": True
}
```

---

## 工具选择

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    tools=tools,
    tool_choice={"type": "tool", "name": "get_weather"},  # 强制使用特定工具
    messages=[{"role": "user", "content": "What's the weather in Paris?"}]
)
```

---

## 代码执行

### 基本用法

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    messages=[{
        "role": "user",
        "content": "Calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"
    }],
    tools=[{
        "type": "code_execution_20260120",
        "name": "code_execution"
    }]
)

for block in response.content:
    if block.type == "text":
        print(block.text)
    elif block.type == "bash_code_execution_tool_result":
        print(f"stdout: {block.content.stdout}")
```

### 上传文件进行分析

```python
# 1. 上传文件
uploaded = client.beta.files.upload(file=open("sales_data.csv", "rb"))

# 2. 通过 container_upload 块传递给代码执行
# 代码执行是 GA；Files API 仍然是 beta（通过 extra_headers 传递）
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    extra_headers={"anthropic-beta": "files-api-2025-04-14"},
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Analyze this sales data. Show trends and create a visualization."},
            {"type": "container_upload", "file_id": uploaded.id}
        ]
    }],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)
```

### 检索生成的文件

```python
import os

OUTPUT_DIR = "./claude_outputs"
os.makedirs(OUTPUT_DIR, exist_ok=True)

for block in response.content:
    if block.type == "bash_code_execution_tool_result":
        result = block.content
        if result.type == "bash_code_execution_result" and result.content:
            for file_ref in result.content:
                if file_ref.type == "bash_code_execution_output":
                    metadata = client.beta.files.retrieve_metadata(file_ref.file_id)
                    file_content = client.beta.files.download(file_ref.file_id)
                    # 使用 basename 防止路径遍历；验证结果
                    safe_name = os.path.basename(metadata.filename)
                    if not safe_name or safe_name in (".", ".."):
                        print(f"Skipping invalid filename: {metadata.filename}")
                        continue
                    output_path = os.path.join(OUTPUT_DIR, safe_name)
                    file_content.write_to_file(output_path)
                    print(f"Saved: {output_path}")
```

### 容器重用

```python
# 第一次请求：设置环境
response1 = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=4096,
    messages=[{"role": "user", "content": "Install tabulate and create data.json with sample data"}],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)

# 从响应中获取容器 ID
container_id = response1.container.id

# 第二次请求：重用同一个容器
response2 = client.messages.create(
    container=container_id,
    model="{{OPUS_ID}}",
    max_tokens=4096,
    messages=[{"role": "user", "content": "Read data.json and display as a formatted table"}],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)
```

### 响应结构

```python
for block in response.content:
    if block.type == "text":
        print(block.text)  # Claude 的解释
    elif block.type == "server_tool_use":
        print(f"Running: {block.name} - {block.input}")  # Claude 正在做什么
    elif block.type == "bash_code_execution_tool_result":
        result = block.content
        if result.type == "bash_code_execution_result":
            if result.return_code == 0:
                print(f"Output: {result.stdout}")
            else:
                print(f"Error: {result.stderr}")
        else:
            print(f"Tool error: {result.error_code}")
    elif block.type == "text_editor_code_execution_tool_result":
        print(f"File operation: {block.content}")
```

---

## 内存工具

### 基本用法

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=2048,
    messages=[{"role": "user", "content": "Remember that my preferred language is Python."}],
    tools=[{"type": "memory_20250818", "name": "memory"}],
)
```

### SDK 内存辅助器

子类化 `BetaAbstractMemoryTool`：

```python
from anthropic.lib.tools import BetaAbstractMemoryTool

class MyMemoryTool(BetaAbstractMemoryTool):
    def view(self, command): ...
    def create(self, command): ...
    def str_replace(self, command): ...
    def insert(self, command): ...
    def delete(self, command): ...
    def rename(self, command): ...

memory = MyMemoryTool()

# 与工具运行器一起使用
runner = client.beta.messages.tool_runner(
    model="{{OPUS_ID}}",
    max_tokens=2048,
    tools=[memory],
    messages=[{"role": "user", "content": "Remember my preferences"}],
)

for message in runner:
    print(message)
```

有关完整的实现示例，请使用 WebFetch：

- `https://github.com/anthropics/anthropic-sdk-python/blob/main/examples/memory/basic.py`

---

## 结构化输出

### JSON 输出（Pydantic — 推荐）

```python
from pydantic import BaseModel
from typing import List
import anthropic

class ContactInfo(BaseModel):
    name: str
    email: str
    plan: str
    interests: List[str]
    demo_requested: bool

client = anthropic.Anthropic()

response = client.messages.parse(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": "Extract: Jane Doe (jane@co.com) wants Enterprise, interested in API and SDKs, wants a demo."
    }],
    output_format=ContactInfo,
)

# response.parsed_output 是一个经过验证的 ContactInfo 实例
contact = response.parsed_output
print(contact.name)           # "Jane Doe"
print(contact.interests)      # ["API", "SDKs"]
```

### 原始 Schema

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": "Extract info: John Smith (john@example.com) wants the Enterprise plan."
    }],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "email": {"type": "string"},
                    "plan": {"type": "string"},
                    "demo_requested": {"type": "boolean"}
                },
                "required": ["name", "email", "plan", "demo_requested"],
                "additionalProperties": False
            }
        }
    }
)

import json
# output_config.format 保证第一个块是带有有效 JSON 的文本
text = next(b.text for b in response.content if b.type == "text")
data = json.loads(text)
```

### 严格工具使用

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Book a flight to Tokyo for 2 passengers on March 15"}],
    tools=[{
        "name": "book_flight",
        "description": "Book a flight to a destination",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "destination": {"type": "string"},
                "date": {"type": "string", "format": "date"},
                "passengers": {"type": "integer", "enum": [1, 2, 3, 4, 5, 6, 7, 8]}
            },
            "required": ["destination", "date", "passengers"],
            "additionalProperties": False
        }
    }]
)
```

### 同时使用两者

```python
response = client.messages.create(
    model="{{OPUS_ID}}",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Plan a trip to Paris next month"}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "summary": {"type": "string"},
                    "next_steps": {"type": "array", "items": {"type": "string"}}
                },
                "required": ["summary", "next_steps"],
                "additionalProperties": False
            }
        }
    },
    tools=[{
        "name": "search_flights",
        "description": "Search for available flights",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "destination": {"type": "string"},
                "date": {"type": "string", "format": "date"}
            },
            "required": ["destination", "date"],
            "additionalProperties": False
        }
    }]
)
```
