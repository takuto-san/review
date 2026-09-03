---
translation_of: skills/review/SKILL.md
language: zh-CN
runtime: false
---

> [!NOTE]
> 本文档是面向读者的简体中文翻译。运行时使用英文`skills/review/SKILL.md`。

# Review

从`$ARGUMENTS`或自然语言识别目标。`/review:review`审查本地更改，`/review:review 123`审查数字PR编号。PR URL不能作为直接参数，应使用“请审查这个PR URL”之类的自然语言请求。

阶段间的所有输入和输出都使用`agents/README.md`定义的A2A兼容Artifact。编排器验证Artifact名称、media type、schema metadata和必需负载，并将完整Artifact传给下一阶段。接收方读取`parts[0].data`，不得从对话历史重建缺失数据。共享目标表示为`review.target`，审查计划表示为`review.plan`。

## 1. 解析审查目标

自然语言中任意位置包含PR编号或URL时使用Reviewer mode，并通过`gh`获取PR、关联Issue、差异和CI状态。存在多个URL或冲突目标时不得猜测。未指定PR但要求审查本地更改时使用Developer mode。

## 2. 判断是否需要审查

Reviewer mode运行`review:validation:review-needed`，依次检查关闭或合并、草稿、微不足道，以及当前认证用户是否已审查当前head SHA。`should_review`为false时简要报告证据并停止，不运行`small-cls`或其他审查层。Developer mode跳过此验证。

## 3. 收集并整理上下文

运行`review:context:context`。不再把关联Issue作为特权入口，而是从用户指定来源、PR关联产物以及与变更相邻的仓库信息构建有边界的搜索锚点。可用时优先使用MCP只读工具，并将其他工具取得的来源也规范化到MCP Resource兼容的`context.resources`。不得把原始文档传给后续代理，并保留无法解析的引用、来源冲突、权威级别和精确位置。

## 4. 分析Change Scope

运行`review:validation:small-cls`，评估差异统计、Change Group、内聚性和可审查性。结果为`review_blocked`时只继续可靠检查，并标记最终审查未完成。

`small-cls`只判断变更量和Change Group是否给审查者造成过大负担，不判断是否需要审查。

## 5. 生成审查计划

编排器读取`REVIEW.md`，考虑八项质量特性，只把相关关注点转换为PR专属问题，并分配给`mechanical`、`structural`或`contextual`层。为每项分配`RP-001`形式的稳定`review_item_id`，并保留选择理由、辅助层和预期证据。每个审查代理必须对每个分配项返回一个结果；证据不足时返回`insufficient_evidence`，不得省略。

## 6. 运行三层审查

并行运行：

- `review:review:mechanical`
- `review:review:structural`
- `review:review:contextual`

向机械层提供CI，向结构层提供差异和代码库，向上下文层提供收集后的上下文。未经明确批准，不执行外部或不可信PR中的代码。

每次委派都必须显式提供仓库根目录、审查目标、base和head SHA、变更文件、完整差异或其明确位置、分配项以及代理特定的必需输入。不得假设subagent能从父对话恢复编排状态。

## 7. 验证审查结果

向`review:comment:comment`传递收集后的上下文、Change Scope、计划、全部结果、命令和`REVIEW.md`。重新验证失败路径和证据，排除推测、既有问题和重复项。只有验证结果可进入最终报告。计划中的每个`review_item_id`必须归入验证结果、拒绝结果或明确的未完成原因，不得静默消失。

## 8. 生成最终报告

编排器只输出Review Summary、Change Scope、Needs Your Attention和Review Coverage。不得创造新Finding或显示内部代理名称和中间YAML。最终决定属于人工审查者。

目标、上下文、范围、计划、审查层、必要静态分析和单元测试或Finding验证缺失时，必须说明原因并标记为未完成。
