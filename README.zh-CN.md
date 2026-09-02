# Review Plugin

[English](README.md) | [日本語](README.ja.md) | 简体中文

使用专业代理、仓库检查和基于证据的Finding验证，自动审查本地更改和GitHub Pull Request。

## 目录

- [1. 概述](#1-overview)
- [2. 架构](#2-architecture)
  - [目录结构](#directory-structure)
  - [审查工作流](#review-workflow)
- [3. 用法](#3-usage)
- [4. 安装](#4-installation)
- [5. 使用指南](#5-usage-guide)
- [6. 配置](#6-configuration)
- [7. 技术细节](#7-technical-details)
- [8. 项目信息](#8-project-information)

<a id="1-overview"></a>
## 1. 概述

Review Plugin收集相关上下文，判断变更是否适合审查，生成针对当前变更的计划，并从mechanical、structural和contextual三个角度审查实现。独立验证阶段会在生成最终报告前排除推测性或重复的Finding。

支持两种模式：

- **Developer mode**：审查当前仓库中的提交和工作树更改。
- **Reviewer mode**：审查通过编号或URL指定的GitHub Pull Request。

本插件是参考常见多阶段审查流程完成的洁净室实现。上下文收集、计划、审查、Finding验证和报告生成均在本插件内完成。

<a id="2-architecture"></a>
## 2. 架构

<a id="directory-structure"></a>
### 目录结构

```text
review/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── context/
│   │   └── context.md
│   ├── validation/
│   │   ├── review-needed.md
│   │   └── small-cls.md
│   ├── review/
│   │   ├── mechanical.md
│   │   ├── structural.md
│   │   └── contextual.md
│   ├── comment/
│   │   └── comment.md
│   └── README.md
├── skills/
│   └── review/
│       └── SKILL.md
├── docs/
│   ├── ja/
│   └── zh-CN/
├── REVIEW.md
├── README.md
├── README.ja.md
├── README.zh-CN.md
└── LICENSE
```

<a id="review-workflow"></a>
### 审查工作流

```mermaid
%%{init: {"flowchart": {"curve": "stepAfter"}}}%%
flowchart TD
    A[解析本地更改或PR] --> B{PR是否需要审查}
    B -->|否| X[报告跳过原因]
    B -->|是或本地更改| C[收集相关上下文]
    C --> D[生成Evidence Packet]
    D --> E[评估审查者负担]
    E --> F[生成变更专属审查计划]
    F --> G1[Mechanical review]
    F --> G2[Structural review]
    F --> G3[Contextual review]
    G1 --> H[验证Finding并去重]
    G2 --> H
    G3 --> H
    H --> I[生成最终报告]
```

<a id="3-usage"></a>
## 3. 用法

### 命令：`/review:review`

基于证据审查本地更改或GitHub Pull Request。

处理内容：

1. 解析本地更改或指定的Pull Request。
2. 在Reviewer mode中判断是否需要审查，并跳过closed、draft、trivial或already-reviewed的PR。
3. 仅收集与审查目标明确相关的上下文。
4. 评估变更是否给审查者造成过大负担。
5. 根据变更和`REVIEW.md`生成审查计划。
6. 并行运行三个专业审查层：
   - Mechanical review：测试、静态分析和客观信号
   - Structural review：设计、执行路径、状态、安全、性能和可维护性
   - Contextual review：需求、意图、兼容性和文档
7. 根据代码和可用证据重新验证Finding候选项。
8. 仅报告已验证的Finding、需要人工判断的事项和明确限制。

用法：

```text
/review:review
/review:review <PR编号>
```

也支持自然语言请求：

```text
审查我的本地更改
审查PR 123
审查这个PR: https://github.com/owner/repository/pull/123
```

Pull Request URL可以用于自然语言请求，但不能作为`/review:review`的直接参数。

### 功能

- Developer mode和Reviewer mode
- Pull Request审查必要性验证
- 基于ISO/IEC 25010质量特性的变更专属计划
- 并行执行Mechanical、Structural和Contextual review
- 从明确引用的来源中进行只读上下文收集
- 使用精简的Evidence Packet而非原始文档
- 独立验证Finding并去重
- 明确展示审查覆盖范围、证据和限制

### 审查报告格式

```text
## Review Summary

Potential problems: 1
Human decisions: 1
Verified concerns: 6
Could not verify: 1

## Change Scope

Focused — one self-contained change

## Needs Your Attention

1. Potential problem: retry can duplicate the write operation
   Evidence: src/example.ts:42
   Confirm: whether the external operation is idempotent

## Review Coverage

| Subcharacteristic | Concern | Result | Evidence |
|---|---|---|---|
| Recoverability | Recovery and consistency | Retry path inspected | src/example.ts:35 |
```

#### 结果分类

- **Potential problem**：变更代码、现实触发条件和可观察影响表明可能存在缺陷。
- **Human decision**：代码事实已知，但需要产品、设计或业务判断。
- **Verified by AI**：已检查适用范围，未发现该关注点的问题。
- **Could not verify**：缺少所需规格、测量结果、权限或执行证据。

#### 过滤的误报

- 未受本次变更实质影响的既有问题
- 没有现实执行路径的假设问题
- 已由CI覆盖的格式、lint或简单类型错误
- 个人风格偏好和含糊的一般建议
- 重复或缺乏依据的Finding

<a id="4-installation"></a>
## 4. 安装

### 前置条件

运行审查需要以下环境和访问权限：

- 支持插件和代理的Claude Code
- Developer mode使用的Git仓库
- Reviewer mode使用的、已安装并认证的GitHub CLI（`gh`）
- 对目标仓库和Pull Request的访问权限
- 用于Mechanical verification的仓库既有测试或分析命令
- 需要外部证据时可用的兼容只读工具

开发时直接加载插件：

```bash
claude --plugin-dir /path/to/review
```

使用前进行验证：

```bash
claude plugin validate /path/to/review --strict
```

<a id="5-usage-guide"></a>
## 5. 使用指南

### 使用`/review:review`的最佳实践

- Pull Request说明应聚焦于意图、行为和约束。
- 明确链接相关Issue、规格和决策。
- 从干净且有效的Git仓库运行审查。
- 将Finding作为人工决策的证据，而不是自动批准或拒绝。
- 保持仓库定义的测试和静态分析命令与CI一致。

#### 适用场景

- 创建Pull Request前的本地更改
- 包含有意义的行为或架构变更的Pull Request
- 涉及关键代码路径、持久化、认证或外部服务的变更
- 需要根据链接来源检查需求或兼容性的变更
- 需要独立验证行为和代码库影响的重构

#### 不适用场景

- 没有可审查的提交或工作树更改
- 只有自动化已覆盖的格式变更或生成文件
- 用于替代缺失的产品需求或人工判断
- 无法通过可用的只读工具访问目标

### 工作流集成

#### 标准本地审查工作流

```text
# 在本地仓库中进行更改
/review:review

# 查看Needs Your Attention和Review Coverage
# 修复已确认的问题并重新运行审查
```

#### 标准Pull Request审查工作流

```text
# 使用PR编号审查
/review:review 123

# 或通过自然语言指定URL
审查这个PR: https://github.com/owner/repository/pull/123

# 确认Finding并由人工作出最终审查决定
```

审查默认是只读的。除非用户另行明确要求，否则不会修改源文件、安装依赖、修改仓库配置或发布GitHub评论。

<a id="6-configuration"></a>
## 6. 配置

### 自定义审查标准

编辑`REVIEW.md`可更改质量关注点、适用条件、验证指南、结果分类或最终报告策略。

默认覆盖模型考虑以下ISO/IEC 25010质量特性：

- 功能适合性
- 可靠性
- 性能效率
- 易用性
- 安全性
- 兼容性
- 可维护性
- 可移植性

仅选择适用于当前变更的标准。

### 自定义代理

代理职责和输出约定定义在`agents/`目录中：

- `agents/context/context.md` — 上下文收集和Evidence Packet生成
- `agents/validation/review-needed.md` — Pull Request审查必要性
- `agents/validation/small-cls.md` — Change Scope和审查者负担
- `agents/review/mechanical.md` — 客观仓库检查
- `agents/review/structural.md` — 代码和架构分析
- `agents/review/contextual.md` — 意图和需求分析
- `agents/comment/comment.md` — Finding验证和去重

编排和最终报告规则保存在`skills/review/SKILL.md`中。

每个审查计划项都使用`RP-001`形式的唯一`review_item_id`，并从审查层保持到Finding验证。必须显式向每个代理提供必需输入；证据不足的分配项不得省略，应返回`insufficient_evidence`。


<a id="7-technical-details"></a>
## 7. 技术细节

### 代理架构

- 1个eligibility agent判断Pull Request是否需要审查。
- 1个context agent收集明确引用的证据。
- 1个scope agent对内聚性和审查者负担进行分类。
- review skill生成针对目标的审查计划。
- 3个专业代理并行审查Mechanical、Structural和Contextual关注点。
- 1个comment agent验证候选Finding并生成已验证结果集。
- review skill在不新增或重新评估Finding的情况下格式化最终报告。

### 上下文处理

context agent只跟随与审查目标相关联的引用。后续阶段接收精简的Evidence Packet，而不是Notion、Confluence、Google Docs、GitHub、Web或仓库文档的原始内容。缺失信息和来源冲突会明确保留在Packet和最终覆盖报告中。

### GitHub集成

Reviewer mode使用`gh`完成：

- 解析Pull Request元数据、分支和SHA
- 读取变更文件、diff、关联Issue和检查状态
- 在不修改工作树的情况下访问仓库信息

如果需要checkout，工作流会从解析出的head SHA创建隔离的临时worktree，并在证据收集后删除。

<a id="8-project-information"></a>
## 8. 项目信息

### Author

takuto-san

### Version

0.1.0

基于[MIT License](LICENSE)授权。
