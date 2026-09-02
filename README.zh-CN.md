# review

[English](README.md) | [日本語](README.ja.md) | 简体中文

一个用于审查本地更改和GitHub Pull Request的Claude Code插件。

本插件是参考常见多阶段审查流程完成的洁净室实现。上下文收集、审查计划、专业化审查层、Finding验证以及最终报告生成均在本插件内完成。

## 文档

- [简体中文文档](docs/zh-CN/README.md)
- [英文文档](README.md)
- [日本語ドキュメント](docs/ja/README.md)

Claude Code代理读取`skills/`、`agents/`以及根目录`REVIEW.md`中的英文文件。`docs/ja/`和`docs/zh-CN/`仅供读者参考，不会由审查代理加载。

## 工作流程

### Developer mode

审查当前本地仓库中的提交和工作树更改。

```mermaid
flowchart TD
    A[识别本地更改<br/>领先upstream的提交、暂存、未暂存和相关未跟踪文件] --> B[理解本地变更意图<br/>阅读提交信息、仓库指南和明确引用]
    B --> C[只收集必要的背景信息<br/>仅查看本地变更能够明确引用的资料]
    C --> D[整理为审查上下文<br/>需求、约束、未决问题和来源]
    D --> E[确认变更是否可审查<br/>规模、内聚性和可独立拆分的变更组]
    E --> F[决定本次必须检查的内容<br/>只选择相关质量标准]

    F --> G1[运行客观检查<br/>CI、静态分析、类型和测试]
    F --> G2[检查代码结构<br/>逻辑、依赖、状态和失败路径]
    F --> G3[对照实现与变更意图<br/>需求、验收条件和约束]

    G1 --> H[验证审查结果<br/>确认证据、删除重复并排除推测]
    G2 --> H
    G3 --> H
    H --> I[展示审查结果<br/>摘要、变更范围、待确认事项和覆盖范围]
```

### Reviewer mode

审查由PR编号或URL指定的GitHub Pull Request。

```mermaid
flowchart TD
    A[解析Pull Request<br/>标题、说明、分支、差异和CI状态] --> B[理解请求的变更<br/>阅读PR和关联Issue]
    B --> C[只收集必要的背景信息<br/>仅跟踪明确引用的规格和决策资料]
    C --> D[整理为审查上下文<br/>需求、约束、未决问题和来源]
    D --> E[确认PR是否可审查<br/>规模、内聚性和可独立拆分的变更组]
    E --> F[决定本次必须检查的内容<br/>只选择相关质量标准]

    F --> G1[运行客观检查<br/>CI、静态分析、类型和测试]
    F --> G2[检查代码结构<br/>逻辑、依赖、状态和失败路径]
    F --> G3[对照实现与变更意图<br/>需求、验收条件和约束]

    G1 --> H[验证审查结果<br/>确认证据、删除重复并排除推测]
    G2 --> H
    G3 --> H
    H --> I[展示审查结果<br/>摘要、变更范围、待确认事项和覆盖范围]
```

`context`代理可以使用用户环境中任何兼容的只读信息源。它只跟踪与审查目标明确关联的引用，并向审查层传递精简的Evidence Packet，而不是原始文档。

## 目录结构

```text
review/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── context/
│   │   └── context.md
│   ├── validation/
│   │   └── small-cls.md
│   ├── review/
│   │   ├── mechanical.md
│   │   ├── structural.md
│   │   └── contextual.md
│   ├── comment/
│   │   └── comment.md
├── skills/
│   └── review/
│       └── SKILL.md
├── REVIEW.md
├── docs/
│   ├── ja/
│   └── zh-CN/
├── README.md
├── README.ja.md
├── README.zh-CN.md
└── LICENSE
```

## 使用方法

```text
/review:review
/review:review 123
/review:review https://github.com/owner/repository/pull/123
审查我的本地更改
请审查这个PR https://github.com/owner/repository/pull/123
```

除显式运行`/review:review`外，Skill也可以根据自然语言审查请求自动选择。请求中任意位置包含PR编号或URL时使用Reviewer mode；没有PR目标但要求审查当前或本地更改时使用Developer mode。

审查开始前，`context`代理只获取审查目标明确引用的信息，并将其转换为精简且与信息源无关的Evidence Packet。Notion、Confluence、Google Docs、GitHub、Web或仓库文档中的原始内容不会被直接传递给审查代理。

## 开发

```bash
claude plugin validate . --strict
claude --plugin-dir .
```

本项目采用MIT许可证。
