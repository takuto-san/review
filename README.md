# review

A Claude Code plugin for reviewing local changes and GitHub pull requests.

This is a clean-room implementation inspired by multi-stage review workflows.
Its review planning, specialized review layers, finding verification, and
report synthesis are implemented entirely by this plugin's Claude Code agents.

## Structure

```text
review/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── change-scope-analyst.md
│   ├── review-plan-builder.md
│   ├── review/
│   │   ├── mechanical-reviewer.md
│   │   ├── structural-reviewer.md
│   │   └── contextual-reviewer.md
│   ├── comment/
│   │   └── comment.md
│   └── review-synthesizer.md
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

## Development

```bash
claude plugin validate . --strict
claude --plugin-dir .
```

Licensed under the MIT License.
