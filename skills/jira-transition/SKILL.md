---
name: jira-transition
description: 使用 Jira MCP 工具将 Jira 问题转换到新的状态。当你需要更改 Jira 问题的状态、工作流状态或通过可用的转换推进问题时使用。
---

# Jira 状态转换

使用 Jira MCP 的 `transition_issue` 工具来推进 Jira 问题的工作流状态。

## 工作流程

1. **获取问题详情**：使用 `jira_get_issue` 获取问题的当前状态、标题、负责人等信息
2. **获取可用转换**：使用 `jira_get_transitions` 获取问题的可用状态转换
3. **匹配目标状态**：根据用户提供的 `status` 找到对应的 transition ID
4. **执行转换**：调用 `jira_transition_issue`，传入 issue key 和 transition ID

## 参数

- `issue_key`: Jira 问题键（如 OR-6415, PROJ-123）
- `status`: 目标状态名称（如 "已完成"、"Done"、"In Progress"）
- `transition_id`: 可选，直接指定 transition ID（优先级高于 status）

## 自动获取信息

执行时会自动获取：
- 当前问题状态
- 问题标题
- 负责人
- 可用的转换列表
- 状态变更历史（从 changelog）

## 状态历史分析

Skill 会分析问题的 changelog，展示状态流转历史，帮助用户理解当前所在位置。

## 多步骤状态转换

当目标状态不在当前可用转换中时，skill 会尝试自动完成多步骤转换：

1. **方案 A - 自动多步骤转换**
   - 如果用户指定 `target_final_status`（最终目标状态）
   - skill 会查找从当前状态到最终目标所需的所有中间状态
   - 自动逐步执行每个转换

2. **方案 B - 引导式转换**
   - 如果无法自动找到路径，会列出所有可用转换
   - 让用户选择下一步
   - 重复直到达到目标

## 自动多步骤转换示例

```
用户输入：/jira-transition issue_key=OR-6415 target_final_status=Code Completed

Skill 执行流程：
1. 获取当前状态：RD Review
2. 获取所有可用转换
3. 查找路径：RD Review → RD Review Pass → ... → Code Completed
4. 自动执行每一步转换
5. 返回最终结果
```

## 参数（新增）

- `target_final_status`: 最终目标状态名称，skill 会自动完成多步骤转换
- `workflow_path`: 可选，手动指定转换路径（如 "RD Review Pass,Merged,Code Completed"）

## 预定义工作流

### OR 项目（Orderly）完整工作流

```
Backlog → Requirement Review → RD Review → RD In Progress → Code Complete → Ready for Testing
```

**状态名称与 Transition ID 对照**：

| 状态名称 | Transition ID |
|----------|---------------|
| Ready for Requirement Review | 281 |
| Ready for RD Review | 291 |
| Start working (RD In Progress) | 11 |
| Complete the coding (Code Complete) | 251 |
| Ready to test (Ready for Testing) | 51 |

**自动填充字段**（当需要时）：
- `Original Estimate`: 默认填 "1d"
- `QA Testing Days`: 默认填 id "10025" (对应值 1)

**处理必填字段的技巧**：
对于需要必填自定义字段的 transition（如 Start working），需要：
1. 先用 `jira_update_issue` 更新字段
2. 再执行 `jira_transition_issue`

**使用示例**：
```
/jira-transition issue_key=OR-6415 target_final_status=Ready for Testing
```

Skill 会自动：
1. 从当前状态逐步转换到目标状态
2. 在需要时自动填充默认字段值
