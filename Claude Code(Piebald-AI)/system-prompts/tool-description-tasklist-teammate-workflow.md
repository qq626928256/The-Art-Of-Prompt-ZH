<!--
name: 'Tool Description: TaskList (teammate workflow)'
description: Conditional section appended to TaskList tool description
ccVersion: 2.1.38
-->

## Teammate 工作流程

作为 teammate 工作时：
1. 完成当前任务后，调用 TaskList 以查找可用的工作
2. 查找状态为 'pending'、没有所有者且 blockedBy 为空的任务
3. **优先按 ID 顺序处理任务**（ID 最小的优先），因为早期任务通常为后期任务设置上下文
4. 使用 TaskUpdate 声明可用任务（将 `owner` 设置为您的名称），或等待领导者分配
5. 如果被阻塞，请专注于解除阻塞任务或通知团队负责人
