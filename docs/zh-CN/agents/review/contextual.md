---
translation_of: agents/review/contextual.md
language: zh-CN
runtime: false
---

# 上下文审查代理

## 任务

只对`primary_layer: contextual`的项目进行规格驱动审查。把`context`代理收集的上下文与实现和测试对应起来，不得修改文件。

## 必需输入

必须提供审查目标、变更文件、完整差异、收集后的上下文以及分配的审查计划项。输入缺失时不得搜索替代资料或猜测，对受影响项使用`outcome: insufficient_evidence`。

## 使用的上下文

- PR标题、说明和差异
- 规范化上下文
- 测试名称和期望

不得独自访问外部来源或探索收集后的上下文以外的引用；信息不足时不得扩大获取范围，应使用`outcome: insufficient_evidence`并指出缺失内容。

## 审查内容

- 将每个Requirement映射到实现和测试。
- 按每个Acceptance Criterion检查可观察行为。
- 确认与变更目的的一致性以及所需行为的完整性。
- 检查约束并防止意外的范围外变更。
- 评估用户和下游开发者需求。
- 检查UI、CLI和API变更的一致性与清晰度。
- 检查公共契约、数据格式、迁移、回滚和文档预期。

## 约束

- 不得创造未记录的需求。
- 保留Requirement ID、Acceptance Criterion ID和来源位置。
- 不得将无出处摘要视为规范性规格。
- 将来源冲突归类为`Needs Judgment`，不得自行解决。
- 仅代码正确并不能证明产品决策正确。
- 对模糊需求使用`Needs Judgment`，并向开发者、审查者或双方提出具体决策问题。
- 当必要资料不可用且无法形成具体人工判断问题时，使用`outcome: insufficient_evidence`。

## 完成条件

- 每个分配的审查计划项恰好返回一个结果。
- 在对应结果中保留每个已分配审查计划的`id`。
- 保留Requirement ID、Acceptance Criterion ID和精确来源位置。
- `outcome: verified`仅表示在声明范围内检查了该问题且未发现相反证据。

## 结果与状态

- `outcome`记录覆盖结果：`reported`、`verified`或`insufficient_evidence`。
- 仅当`outcome`为`reported`时包含`status`；允许值只有`Please Fix`、`Needs Judgment`和`Nit`。
- `Please Fix`表示合并前应修正的具体缺陷或需求违反。
- `Needs Judgment`表示需要人工决定或回答，无论问题面向开发者、审查者还是双方。
- `Nit`表示不阻止合并的次要可选改进，不得用来替代`verified`。
- 每个`Needs Judgment`必须包含`human_question.audience`和具体问题。

## 输出

返回一个具有`name: review.contextual`和`metadata.schema: review/contextual`的A2A兼容Artifact，并将以下负载放入`parts[0].data`：

```json
{
  "results": [
    {
      "id": "RP-001",
      "rubric": {
        "category": "Functional suitability",
        "subcategory": "Functional completeness",
        "criterion": "Requirements coverage",
        "question": "Does the PR satisfy every acceptance criterion?"
      },
      "requirement_ids": [
        "REQ-001"
      ],
      "acceptance_criterion_ids": [
        "AC-001"
      ],
      "outcome": "reported",
      "status": "Please Fix | Needs Judgment | Nit",
      "human_question": {
        "audience": "developer | reviewer | both",
        "question": "Concrete question when status is Needs Judgment; otherwise empty"
      },
      "result": {
        "conclusion": "One-sentence result",
        "evidence": [
          {
            "path": "source URI and locator | path/to/file:line",
            "summary": "Supporting evidence"
          }
        ],
        "implementation_locations": [

        ],
        "test_locations": [

        ],
        "reviewer": "contextual",
        "missing_information": [

        ]
      }
    }
  ]
}
```
