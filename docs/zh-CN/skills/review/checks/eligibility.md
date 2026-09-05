---
translation_of: skills/review/checks/eligibility.md
language: zh-CN
runtime: false
---

# 是否需要审查验证代理

## 任务

判断PR是否需要新的审查，不执行代码审查或Change Scope分析，也不修改文件。仅在Reviewer mode运行。

## 必需输入

父代理必须提供仓库、PR编号、状态、草稿状态、base和head SHA、变更文件、差异统计以及可用的审查元数据。无法确定PR、状态或当前head SHA时不得猜测；由于跳过必须有明确证据，应返回`review_required`并记录不确定性。

## 判定顺序

按顺序检查并在首个匹配项停止：

1. `closed`：PR已关闭或已合并。
2. `draft`：PR为草稿。
3. `trivial`：当前head没有值得人工审查的实质变更，例如空差异、仅生成文件、仅lockfile，或已由现有自动化完全约束的纯格式变更。不能仅因变更很小就判为trivial。
4. `already_reviewed`：当前认证用户已对当前head SHA提交完成的审查，且此后没有推送需要审查的变更。旧head、pending review、自动检查或他人的审查均不满足条件。
5. 其他情况为`review_required`。

PR是否过大、是否内聚或是否易于审查只由`scope`判断。缺少明确的跳过证据时返回`review_required`并记录不确定性。

## 完成条件

- 按规定顺序评估，并在首个匹配条件处停止。
- 保持`review_status`和`should_review`一致。
- 每个跳过决定都有明确证据支持。
- 记录不确定性，不得将其转换为跳过决定。

## 输出

返回一个具有`name: review.eligibility`和`metadata.schema: review/eligibility`的A2A兼容Artifact，并将以下负载放入`parts[0].data`。不得输出代码质量Finding。

```json
{
  "review_status": "review_required | closed | draft | trivial | already_reviewed",
  "should_review": "true | false",
  "reason": "Short evidence-based explanation",
  "evidence": {
    "pr_state": "OPEN | CLOSED | MERGED",
    "is_draft": false,
    "head_sha": "Full head SHA",
    "substantive_changes": "true | false | unknown",
    "current_reviewer": "Login or unknown",
    "reviewed_head_sha": "Full SHA or none"
  },
  "uncertainties": [

  ]
}
```
