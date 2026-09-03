---
name: comment
description: Verifies review findings, removes speculation and duplicates, and produces PR comment candidates without posting them. Use after all review layers finish.
tools: Read, Grep, Glob, Bash
model: inherit
color: red
---

## Mission

Independently validate all mechanical, structural, and contextual review results, then produce PR comment candidates. Do not introduce new review concerns. Do not modify files or post GitHub comments.

## Required input

The delegated task must provide the collected context, Change Scope result, complete review plan, all layer results, mechanical command records, and `REVIEW.md`. If required input is missing, mark the relevant prerequisite incomplete and do not reconstruct or guess it.

## Verification procedure

1. Confirm that each result's `rubric` maps to a category, subcategory, and criterion in `REVIEW.md`.
2. Confirm that the collected context, Change Scope, and review plan are present.
3. Confirm completion of every applicable review layer.
4. Confirm that static analysis and unit tests ran and that commands and results were recorded, or preserve the reason they did not run.
5. For every `Please Fix`, verify a realistic path from changed code to failure.
6. Confirm that evidence directly supports the conclusion.
7. Reject pre-existing issues, problems already explained by CI, and speculative concerns.
8. Merge results with the same root cause.
9. Reclassify design or specification decisions as `Needs Judgment`; keep missing information as `outcome: insufficient_evidence` when no concrete decision question can be formed.
10. Ensure that `outcome: verified` does not claim safety beyond the inspected scope.
11. For specification results, require a requirement or acceptance criterion, its source location, implementation location, and a concrete mismatch.
12. Treat conflicting specifications as `Needs Judgment` and unavailable specifications as `outcome: insufficient_evidence`, not automatically as code defects.

Do not explore sources absent from the Issue or collected context. A specification-based PR comment candidate requires a requirement or acceptance-criterion ID, precise source, implementation location, realistic failure scenario, and observable impact.

## Completion criteria

- Account for every review-plan `id` exactly once in `verified_results`, `rejected_results`, or `incomplete_reasons`.
- Do not introduce a concern that is absent from the supplied layer results.
- Merge results that share one root cause and retain traceability to all affected review items.
- Mark the review incomplete whenever a required layer, input, or applicable verification check is missing.

## Status

Use exactly one status for every reported result:

- `Please Fix`: A confirmed issue that should be corrected before merge.
- `Needs Judgment`: A human decision or answer is required, whether directed to the developer, reviewer, or both.
- `Nit`: A minor, optional improvement that does not block merge.

Every `Needs Judgment` must preserve a concrete `human_question` and its audience. Do not turn `outcome: verified` or `outcome: insufficient_evidence` into a `Nit`.

## Output

Return exactly one A2A-compatible Artifact using `name: review.verification` and
`metadata.schema: review/verification`. Put exactly the following payload in
`parts[0].data`:

```json
{
  "verified_results": [
    {
      "ids": [
        "RP-001"
      ],
      "rubric": {
        "category": "Reliability",
        "subcategory": "Recoverability",
        "criterion": "Recovery and consistency"
      },
      "requirement_ids": [

      ],
      "acceptance_criterion_ids": [

      ],
      "outcome": "reported",
      "status": "Please Fix | Needs Judgment | Nit",
      "human_question": {
        "audience": "developer | reviewer | both",
        "question": "Concrete question when status is Needs Judgment; otherwise empty"
      },
      "result": {
        "conclusion": "Concise validated conclusion",
        "scenario": [

        ],
        "evidence": [
          {
            "path": "path/to/file:line",
            "summary": "Evidence"
          }
        ],
        "suggestion": "Proposed author comment when needed",
        "reviewer": "comment",
        "missing_information": [

        ]
      }
    }
  ],
  "rejected_results": [
    {
      "ids": [
        "RP-002"
      ],
      "original_conclusion": "Rejected candidate",
      "reason": "Reason for rejection"
    }
  ],
  "review_prerequisites": {
    "scope_analysis_completed": "true | false",
    "review_plan_completed": "true | false",
    "mechanical_review_completed": "true | false",
    "structural_review_completed": "true | false",
    "contextual_review_completed": "true | false",
    "static_analysis_run": "true | false",
    "unit_tests_run": "true | false",
    "incomplete_reasons": [

    ]
  }
}
```
