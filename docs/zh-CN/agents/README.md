---
translation_of: agents/README.md
language: zh-CN
runtime: false
---

# PR审查代理

| 代理 | 职责 |
|---|---|
| `context` | 通过有边界的发现收集影响判断的信息，生成与来源无关的精简上下文 |
| `review-needed` | 跳过已关闭、草稿、微不足道或已审查的PR |
| `small-cls` | 评估规模、Change Group和内聚性是否造成过大审查负担 |
| `mechanical` | 运行CI等效测试、静态分析和客观检查 |
| `structural` | 审查设计、执行路径、状态、性能、安全和可维护性 |
| `contextual` | 以规格驱动方式审查需求、意图、兼容性和文档 |
| `comment` | 重新验证Finding，删除推测和重复，生成PR评论候选 |

推荐顺序：

1. Reviewer mode下的`review-needed`
2. `context`
3. `small-cls`
4. `skills/review/SKILL.md`生成审查计划
5. `mechanical`、`structural`和`contextual`
6. `comment`
7. `skills/review/SKILL.md`生成最终报告

机械检查和两个审查代理可以并行运行。

## 代理产物契约

每个代理返回一个兼容A2A的`Artifact` JSON对象。如果代理专用输出示例不包含`artifactId`，该示例仅表示放入`parts[0].data`的负载。

```json
{
  "artifactId": "context-<target-id>",
  "name": "review.context",
  "parts": [
    {
      "mediaType": "application/json",
      "data": {}
    }
  ],
  "metadata": {
    "schema": "review/context",
    "schemaVersion": "1.0",
    "producer": "review:context"
  }
}
```

各类数据和阶段使用`review.target`、`review.eligibility`、`review.context`、`review.scope`、`review.plan`、`review.mechanical`、`review.structural`、`review.contextual`和`review.verification`。编排器通过这些Artifact信封传递必需输入，接收方从`parts[0].data`读取类型化负载。不得根据对话历史推测缺失字段。

## 完成要求

- 每个审查计划项都有稳定的`review_item_id`，并在审查和验证过程中保持不变。
- 每个代理结果都使用共同的A2A兼容Artifact信封。
- 必须显式向每个代理提供所需输入；代理不得从父对话推断编排状态。
- 阶段间输入和输出使用共同的A2A兼容Artifact信封。
- 结构和上下文审查代理对每个分配项恰好返回一个结果，证据不足时使用`insufficient_evidence`，不得省略。
- `mechanical`必须在安全且适用时运行仓库定义的静态分析和单元测试。
- 必须记录每条已执行的验证命令及其结果。
- `comment`必须验证各层和检查是否完成，不得把未完成审查视为完成。
