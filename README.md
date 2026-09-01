# review

A Claude Code plugin for reviewing local changes and GitHub pull requests.

This is a clean-room implementation inspired by common multi-stage review
workflows. It calls Claude Code's official code-review capability as a
dependency and does not copy or redistribute its implementation.

## Structure

```text
review/
├── .claude-plugin/
│   └── plugin.json
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
