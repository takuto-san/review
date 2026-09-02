# review

A Claude Code plugin for reviewing local changes and GitHub pull requests.

This is a clean-room implementation inspired by multi-stage review workflows.
Its review planning, specialized review layers, finding verification, and
report synthesis are implemented entirely by this plugin's Claude Code agents.

## Workflow

```mermaid
flowchart TD
    A[Review target<br/>Local changes or Pull Request] --> B[Resolve PR metadata and related Issues]
    B --> C[context/context.md<br/>Collect only explicitly referenced information]
    C --> D[Source-independent Evidence Packet]
    D --> E[validation/small-cls.md<br/>Validate scope, cohesion, and reviewability]
    E --> F[commands/review.md<br/>Build a PR-specific review plan]

    F --> G1[Mechanical review<br/>CI, static analysis, types, and tests]
    F --> G2[Structural review<br/>Design, execution paths, state, and interfaces]
    F --> G3[Contextual review<br/>Requirements, acceptance criteria, and constraints]

    G1 --> H[comment/comment.md<br/>Verify evidence, classify, and deduplicate findings]
    G2 --> H
    G3 --> H

    H --> I[commands/review.md<br/>Produce the final reviewer report]
    I --> J[Review Summary<br/>Change Scope<br/>Needs Your Attention<br/>Review Coverage]
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
│   │   ├── mechanical-reviewer.md
│   │   ├── structural-reviewer.md
│   │   └── contextual-reviewer.md
│   ├── comment/
│   │   └── comment.md
├── commands/
│   └── review.md
├── REVIEW.md
├── README.md
└── LICENSE
```

## Usage

```text
/review:review
/review:review 123
/review:review https://github.com/owner/repository/pull/123
```

The first command reviews local branch and working-tree changes. Supplying a PR
number or URL switches to reviewer mode.

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
