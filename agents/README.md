# PR review agents

| Agent | Responsibility |
|---|---|
| `context` | Uses bounded discovery to collect decision-shaping information and produces compact, source-independent context |
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

The mechanical checks and the two review agents can run in parallel.

## Agent artifact contract

Every agent returns one A2A-compatible `Artifact` JSON object. Unless an
agent-specific example includes `artifactId`, it shows only the payload placed
in `parts[0].data`.

```json
{
  "artifactId": "context-<target-id>",
  "name": "review.context",
  "parts": [
    {
      "mediaType": "application/json",
      "data": {}
    }
  ],
  "metadata": {
    "schema": "review/context",
    "schemaVersion": "1.0",
    "producer": "review:context"
  }
}
```

Use `review.target`, `review.eligibility`, `review.context`, `review.scope`,
`review.plan`, `review.mechanical`, `review.structural`, `review.contextual`,
and `review.verification` as artifact names for the corresponding data and
stages. The orchestrator passes required inputs in these Artifact envelopes,
and each receiver reads the typed payload from `parts[0].data`. No stage may
infer missing payload fields from conversation history.

## Completion requirements

- Every review-plan item has a stable `id` that is preserved through review and verification.
- Every agent result uses the shared A2A-compatible Artifact envelope.
- Every agent receives its required inputs explicitly; agents do not infer orchestration state from the parent conversation.
- Inter-stage inputs and outputs use the shared A2A-compatible Artifact envelope.
- Each structural and contextual review agent returns exactly one result per assigned item, using `assessment.evaluation.level: not_assessable` and `assessment.missing_information` instead of omission when evidence is insufficient.
- `mechanical` must run repository-defined static analysis and unit tests when safe and applicable.
- Every executed verification command and result must be recorded.
- `comment` must verify layer and check completion and must not treat an incomplete review as complete.

## Review evaluation and batching

Structural and contextual work is split into batches of at most five related items per invocation (prefer three to five; smaller batches are valid). The orchestrator assigns unique batch Artifact IDs and consolidates results with exactly one result per assigned item before verification. Three review layers can therefore require more than three agent invocations.

Conformance has four levels plus a separate `not_assessable` state. The verification agent checks evidence and maps evaluations to the five workflow labels defined in `agents/comment/comment.md`. Labels and suggested fixes support human triage; they do not automatically authorize author requests or merge decisions.
