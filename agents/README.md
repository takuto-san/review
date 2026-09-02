# PR review agents

| Agent | Responsibility |
|---|---|
| `context` | Collects only explicitly referenced information and produces a source-independent Evidence Packet |
| `small-cls` | Evaluates size, Change Groups, cohesion, and reviewability |
| `mechanical` | Runs CI-equivalent tests, static analysis, and other objective checks |
| `structural` | Reviews design, execution paths, state, performance, security, and maintainability |
| `contextual` | Performs specification-driven review of requirements, intent, compatibility, and documentation |
| `comment` | Revalidates findings, removes speculation and duplicates, and produces PR comment candidates |

Recommended order:

1. `context`
2. `small-cls`
3. `skills/review/SKILL.md` builds the review plan
4. `mechanical`, `structural`, and `contextual`
5. `comment`
6. `skills/review/SKILL.md` produces the final report

The three review agents can evaluate their assigned items in parallel.

## Completion requirements

- `mechanical` must run repository-defined static analysis and unit tests when safe and applicable.
- Every executed verification command and result must be recorded.
- `comment` must verify layer and check completion and must not treat an incomplete review as complete.
