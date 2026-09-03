# PR review agents

| Agent | Responsibility |
|---|---|
| `context` | Collects only explicitly referenced information and produces compact, source-independent context |
| `review-needed` | Checks PR review eligibility and skips closed, draft, trivial, or already-reviewed PRs |
| `small-cls` | Evaluates whether size, Change Groups, and cohesion create excessive reviewer workload |
| `mechanical` | Runs CI-equivalent tests, static analysis, and other objective checks |
| `structural` | Reviews design, execution paths, state, performance, security, and maintainability |
| `contextual` | Performs specification-driven review of requirements, intent, compatibility, and documentation |
| `comment` | Revalidates findings, removes speculation and duplicates, and produces PR comment candidates |

Recommended order:

1. `review-needed` in Reviewer mode
2. `context`
3. `small-cls`
4. `skills/review/SKILL.md` builds the review plan
5. `mechanical`, `structural`, and `contextual`
6. `comment`
7. `skills/review/SKILL.md` produces the final report

The three review agents can evaluate their assigned items in parallel.

## Completion requirements

- Every review-plan item has a stable `id` that is preserved through review and verification.
- Every agent receives its required inputs explicitly; agents do not infer orchestration state from the parent conversation.
- Each review agent returns exactly one result per assigned item, using `insufficient_evidence` instead of omission.
- `mechanical` must run repository-defined static analysis and unit tests when safe and applicable.
- Every executed verification command and result must be recorded.
- `comment` must verify layer and check completion and must not treat an incomplete review as complete.
