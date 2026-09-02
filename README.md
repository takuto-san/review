# review

English | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

A Claude Code plugin for reviewing local changes and GitHub pull requests.

This is a clean-room implementation inspired by multi-stage review workflows.
Its context collection, review planning, specialized review layers, finding
verification, and report generation are implemented entirely by this plugin.

## Documentation

- [Review criteria](REVIEW.md)
- [日本語ドキュメント](docs/ja/README.md)
- [简体中文文档](docs/zh-CN/README.md)

Claude Code loads the English files in `skills/`, `agents/`, and the root
`REVIEW.md`. Files under `docs/ja/` and `docs/zh-CN/` are human-readable
translations and are not loaded by the review agents.

## Workflow

### Developer mode

Reviews commits and working-tree changes in the current local repository.

```mermaid
flowchart TD
    A[Find local changes<br/>Commits ahead of upstream, staged, unstaged, and relevant untracked files] --> B[Determine the local change intent<br/>Read commit messages, repository guidance, and explicit references]
    B --> C[Gather only the needed background<br/>Follow references available from the local change]
    C --> D[Create a concise review context<br/>Requirements, constraints, open questions, and sources]
    D --> E[Check whether the change is reviewable<br/>Size, cohesion, and independent change groups]
    E --> F[Decide what this review must check<br/>Select relevant quality criteria]

    F --> G1[Run objective checks<br/>CI, static analysis, types, and tests]
    F --> G2[Inspect the code structure<br/>Logic, dependencies, state, and failure paths]
    F --> G3[Compare implementation with intent<br/>Requirements, acceptance criteria, and constraints]

    G1 --> H[Validate the review findings<br/>Confirm evidence, remove duplicates, and reject speculation]
    G2 --> H
    G3 --> H
    H --> I[Present the review result<br/>Summary, scope, items needing attention, and coverage]
```

### Reviewer mode

Reviews a GitHub Pull Request identified by its number or URL.

```mermaid
flowchart TD
    A[Resolve the Pull Request<br/>Title, description, branches, diff, and CI status] --> B[Understand the requested change<br/>Read the PR and related Issues]
    B --> C[Gather only the needed background<br/>Follow explicit links to specifications and decisions]
    C --> D[Create a concise review context<br/>Requirements, constraints, open questions, and sources]
    D --> E[Check whether the PR is reviewable<br/>Size, cohesion, and independent change groups]
    E --> F[Decide what this review must check<br/>Select relevant quality criteria]

    F --> G1[Run objective checks<br/>CI, static analysis, types, and tests]
    F --> G2[Inspect the code structure<br/>Logic, dependencies, state, and failure paths]
    F --> G3[Compare implementation with intent<br/>Requirements, acceptance criteria, and constraints]

    G1 --> H[Validate the review findings<br/>Confirm evidence, remove duplicates, and reject speculation]
    G2 --> H
    G3 --> H
    H --> I[Present the review result<br/>Summary, scope, items needing attention, and coverage]
```

The context agent may use any compatible read-only source available in the
user's environment. It follows only references connected to the review target
and passes a compact Evidence Packet—not raw source documents—to the review
layers.

## Structure

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

## Usage

```text
/review:review
/review:review 123
/review:review https://github.com/owner/repository/pull/123
Review my local changes
Review this PR: https://github.com/owner/repository/pull/123
```

The review Skill can be invoked with `/review:review` or selected automatically
from a natural-language review request. A PR number or URL anywhere in the
request selects Reviewer mode; a request to review current or local changes
without a PR target selects Developer mode.

Before review begins, the `context` agent follows only references explicitly
connected to the review target and converts the required information into a
compact, source-independent Evidence Packet. Raw Notion, Confluence, Google
Docs, GitHub, web, or repository content is not passed directly to the review
agents.

## Development

```bash
claude plugin validate . --strict
claude --plugin-dir .
```

Licensed under the MIT License.
