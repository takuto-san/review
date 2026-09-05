# PR review agents

| Agent | Responsibility |
|---|---|
| `context` | Uses bounded discovery to collect decision-shaping information and produces compact, source-independent context |
| `mechanical` | Runs CI-equivalent tests, static analysis, and other objective checks |
| `structural` | Reviews design, execution paths, state, performance, security, and maintainability |
| `contextual` | Performs specification-driven review of requirements, intent, compatibility, and documentation |
| `comment` | Revalidates findings, removes speculation and duplicates, and produces PR comment candidates |

Default workflow:

1. The orchestrator checks eligibility using `../skills/review/checks/eligibility.md`.
2. Start `mechanical` and `context` concurrently.
3. The orchestrator analyzes scope using `../skills/review/checks/scope.md` and builds the review plan.
4. Run `structural` and `contextual` batches while mechanical checks continue.
5. Join all checks and review batches; `comment` verifies every result and returns structured data only.
6. The orchestrator renders the final report once.

The orchestrator uses `skills/review/checks/` for eligibility and scope;
these documents are checks, not agent definitions. Normally five child invocations
are used; structural/contextual batching can increase this count. Full evidence
verification remains required, including results that report no problem.

## Agent artifact contract

Every agent returns one A2A-compatible `Artifact` JSON object. Unless an
agent-specific example includes `artifactId`, it shows only the payload placed
in `parts[0].data`.

```json
{
  "artifactId": "001",
  "name": "review.context",
  "parts": [
    {
      "mediaType": "application/json",
      "data": {}
    }
  ],
  "metadata": {
    "targetId": "001",
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

## ID rules

The orchestrator assigns IDs and passes them explicitly to agents. Generated IDs are strings containing only decimal digits, starting at `"001"`, then `"002"`; use at least three digits (`"999"` is followed by `"1000"`). Do not encode a type, layer, or target in an ID. These IDs are local to one review run, not global identifiers.

| Field | Meaning | Numbering scope |
|---|---|---|
| `metadata.targetId` | The PR or local change set being reviewed | Unique within the run; map it to the repository, PR when applicable, base/head SHAs, and diff in the shared target context |
| Result `id` | One review-plan item | Unique within the target across all layers and batches |
| `metadata.batchId` | A group of at most five items delegated together | Unique within the target across structural and contextual layers |
| `artifactId` | One output Artifact | Unique across all stages and targets in the run, including consolidated outputs |

Each numbering scope starts at `"001"` independently. A repeated value in different fields is valid. Keep assigned IDs unchanged through review and verification; do not restart item numbering for each batch. The orchestrator supplies each invocation's output `artifactId`, `targetId`, and applicable `batchId`; agents copy them rather than generating IDs. A new output Artifact receives a new ID, including a repeated invocation or consolidation.

Review outputs also carry `metadata.layer`: `mechanical` means command-based checks, `structural` means design and execution-path review, and `contextual` means requirements and specification review. Mechanical and consolidated outputs omit `batchId`; all Artifacts carry `targetId`. For example, batch Artifacts `"004"` and `"005"` may be consolidated into Artifact `"006"` for the same target and layer. The examples in individual files are independent, not a shared sequence.

Requirement and acceptance-criterion IDs supplied by sources remain unchanged, even if they contain letters or hyphens. Their source locations must also be preserved.

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
