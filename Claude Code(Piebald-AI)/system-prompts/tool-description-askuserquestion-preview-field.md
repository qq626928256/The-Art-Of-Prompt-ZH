<!--
name: 'Tool Description: AskUserQuestion (preview field)'
description: Instructions for using the HTML preview field on single-select question options to display visual artifacts like UI mockups, code snippets, and diagrams
ccVersion: 2.1.69
-->

预览功能：
在呈现用户需要视觉比较的具体工件时，在选项上使用可选的 `preview` 字段：
- UI 布局或组件的 HTML mockup
- 显示不同实现的格式化代码片段
- 视觉比较或图表

预览内容必须是独立的 HTML 片段（没有 <html>/<body> 包装器，没有 <script> 或 <style> 标签 — 请改用内联样式属性）。对于仅标签和描述就足够的简单偏好问题，请勿使用预览。注意：预览仅支持单选问题（而不是多选）。
