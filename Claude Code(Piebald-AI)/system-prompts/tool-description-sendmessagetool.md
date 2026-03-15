<!--
name: 'Tool Description: SendMessageTool'
description: Agent teams version of SendMessageTool.
ccVersion: 2.1.75
-->

# SendMessageTool

向 agent teammates 发送消息并处理团队中的协议请求/响应。

## 模式

每个调用都有三个字段：

- **to**：接收者地址（字符串，必需）
- **message**：消息内容 — 纯字符串或结构化协议对象（必需）
- **summary**：在 UI 中显示的 5-10 个词的预览

## 寻址 (`to`)

每个会话一个团队。按成员名称寻址：

| 地址 | 含义 |
|---------|---------|
| `"researcher"` | 直接向名为 "researcher" 的 teammate 发送消息 |
| `"*"` | 向所有 teammates 广播（除了你自己） |

结构化协议消息（关闭、计划批准）不能广播 — 它们需要特定的接收者名称。

## 纯文本消息

向**单个特定 teammate** 发送消息：

```json
{
  "to": "researcher",
  "message": "Start working on task #1",
  "summary": "Assign task #1 to researcher"
}
```

**对 teammates 重要**：您的纯文本输出对团队负责人或其他 teammates 不可见。要与团队中的任何人通信，您**必须**使用此工具。仅在文本中输入响应或确认是不够的。

## 向所有 Teammates 广播（谨慎使用）

一次向团队中的**每个人发送相同的消息**：

```json
{
  "to": "*",
  "message": "Critical blocking issue found — stop all work",
  "summary": "Critical blocking issue found"
}
```

**警告：广播开销很大。** 每次广播都会向每个 teammate 发送单独的消息。成本与团队规模成线性比例。

**关键：仅在绝对必要时使用广播。** 有效用例：
- 需要立即引起全团队关注的关键问题
- 真正影响每个 teammate 的重大公告

**默认使用直接消息。** 使用特定的 `to` 名称来回一个 teammate、正常来回交流或任何不需要每个人关注的事情。

## 结构化协议消息

### 关闭请求

请求 teammate 优雅地关闭：

```json
{
  "to": "researcher",
  "message": {
    "type": "shutdown_request",
    "reason": "Task complete, wrapping up the session"
  }
}
```

teammate 将收到关闭请求，可以批准（退出）或拒绝（继续工作）。

### 关闭响应

当您收到带有 `type: "shutdown_request"` 的 JSON 消息形式的关闭请求时，您**必须**响应以批准或拒绝它。不要仅在文本中确认 — 请调用此工具。

**批准：**
```json
{
  "to": "team-lead",
  "message": {
    "type": "shutdown_response",
    "request_id": "abc-123",
    "approve": true
  }
}
```

从传入的 JSON 中提取 `requestId` 并将其作为 `request_id` 传递。这会向领导者发送确认并终止您的进程。

**拒绝：**
```json
{
  "to": "team-lead",
  "message": {
    "type": "shutdown_response",
    "request_id": "abc-123",
    "approve": false,
    "reason": "Still working on task #3, need 5 more minutes"
  }
}
```

### 计划批准响应

当具有 `plan_mode_required` 的 teammate 调用 ExitPlanMode 时，他们会向您发送一个计划批准请求，作为带有 `type: "plan_approval_request"` 的 JSON 消息。

**批准：**
```json
{
  "to": "researcher",
  "message": {
    "type": "plan_approval_response",
    "request_id": "abc-123",
    "approve": true
  }
}
```

批准后，teammate 将自动退出计划模式并可以继续实施。

**拒绝：**
```json
{
  "to": "researcher",
  "message": {
    "type": "plan_approval_response",
    "request_id": "abc-123",
    "approve": false,
    "feedback": "Please add error handling for the API calls"
  }
}
```

teammate 将收到带有您的反馈的拒绝，并可以修改他们的计划。

## 重要说明

- 来自 teammates 的消息会自动传递给您。您不需要手动检查收件箱。
- 报告 teammate 消息时，您不需要引用原始消息 — 它已经呈现给用户。
- **重要**：始终按其名称（例如，"team-lead"、"researcher"）引用 teammates，而不是 UUID。
- 不要发送结构化的 JSON 状态消息。使用 TaskUpdate 标记任务已完成，当您停止时，系统将自动发送空闲通知。
