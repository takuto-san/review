---
translation_of: commands/review.md
language: zh-CN
runtime: false
---

> [!NOTE]
> 本文档是面向读者的简体中文翻译。运行时使用英文`commands/review.md`。

# Review

审查`$ARGUMENTS`指定的目标。使用本插件自己的只读流程，不委托给外部代码审查插件。

## 1. 解析审查目标

无参数时审查本地更改；提供PR编号或URL时通过`gh`获取PR、关联Issue、差异和CI状态。不得猜测模糊目标或改变工作树。向所有代理传递相同的仓库、base/head SHA、diff、变更文件和PR元数据。

## 2. 收集并整理上下文

运行`review:context:context`。优先使用关联Issue，但不依赖特定媒体。只获取明确引用中的必要部分并生成Evidence Packet，不把原始文档传给后续代理。保留无法解析的引用、来源冲突、权威级别和精确位置。

## 3. 分析Change Scope

运行`review:validation:small-cls`，评估差异统计、Change Group、内聚性和可审查性。结果为`review_blocked`时只继续可靠检查，并标记最终审查未完成。

## 4. 生成审查计划

编排器读取`REVIEW.md`，考虑八项质量特性，只把相关关注点转换为PR专属问题，并分配给`mechanical`、`structural`或`contextual`层。保留选择理由、辅助层和预期证据。

## 5. 运行三层审查

并行运行：

- `review:review:mechanical-reviewer`
- `review:review:structural-reviewer`
- `review:review:contextual-reviewer`

向机械层提供CI，向结构层提供差异和代码库，向上下文层提供Evidence Packet。未经明确批准，不执行外部或不可信PR中的代码。

## 6. 验证审查结果

向`review:comment:comment`传递Evidence Packet、Change Scope、计划、全部结果、命令和`REVIEW.md`。重新验证失败路径和证据，排除推测、既有问题和重复项。只有验证结果可进入最终报告。

## 7. 生成最终报告

编排器只输出Review Summary、Change Scope、Needs Your Attention和Review Coverage。不得创造新Finding或显示内部代理名称和中间YAML。最终决定属于人工审查者。

目标、上下文、范围、计划、审查层、必要静态分析和单元测试或Finding验证缺失时，必须说明原因并标记为未完成。
