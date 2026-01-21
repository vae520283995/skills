---
name: jira-transition
description: 使用 Jira MCP 工具将 Jira 问题转换到新的状态。当你需要更改 Jira 问题的状态、工作流状态或通过可用的转换推进问题时使用。
---

# Jira 状态转换

使用 Jira MCP 的 `transition_issue` 工具来推进 Jira 问题的工作流状态。

## 工作流程

1. 使用 `jira_get_transitions` 获取问题的可用转换
2. 找到目标状态对应的 transition ID
3. 调用 `jira_transition_issue`，传入 issue key 和 transition ID

## 参数

- `issue_key`: Jira 问题键（如 OR-6415, PROJ-123）
- `transition_id`: 要执行的转换 ID（从 `jira_get_transitions` 获取）

## 提示

- 先使用 `jira_get_transitions` 查看问题可用的转换
- 某些转换需要额外字段（如 resolution）
- 转换中的状态名称可能与显示名称不同

## 示例

```
获取转换：使用 jira_get_transitions 查询 PROJ-123 的可用转换
转换到"完成"：调用 jira_transition_issue，transition_id 为 "31"
```
