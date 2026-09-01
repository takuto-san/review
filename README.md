# review

A Claude Code plugin for reviewing local changes and GitHub pull requests.

This is an early clean-room implementation. It calls Claude Code's official
code-review capability as a dependency and does not copy its implementation.

## Usage

```text
/review:review
/review:review 123
/review:review https://github.com/owner/repository/pull/123
```

Natural-language requests such as “Review this pull request” can also activate
the skill.

## Development

```bash
claude plugin validate . --strict
claude --plugin-dir .
```

Licensed under the MIT License.
