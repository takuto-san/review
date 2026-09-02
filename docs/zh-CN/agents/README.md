---
translation_of: agents/README.md
language: zh-CN
runtime: false
---

# PR审查代理

| 代理 | 职责 |
|---|---|
| `context` | 仅从明确引用中收集必要信息，生成与来源无关的Evidence Packet |
| `review-needed` | 跳过已关闭、草稿、微不足道或已审查的PR |
| `small-cls` | 评估规模、Change Group和内聚性是否造成过大审查负担 |
| `mechanical` | 运行CI等效测试、静态分析和客观检查 |
| `structural` | 审查设计、执行路径、状态、性能、安全和可维护性 |
| `contextual` | 以规格驱动方式审查需求、意图、兼容性和文档 |
| `comment` | 重新验证Finding，删除推测和重复，生成PR评论候选 |

推荐顺序为Reviewer mode的`review-needed`、`context`、`small-cls`、由`skills/review/SKILL.md`生成计划、三层审查、`comment`和最终报告。三层审查可以并行运行。

机械审查必须在安全且适用时运行静态分析和单元测试，并记录全部命令与结果。`comment`不得把未完成审查视为完成。
