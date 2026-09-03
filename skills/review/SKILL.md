---
name: review
description: Review local code changes or a GitHub Pull Request using review-need validation, context collection, scope validation, three specialized review layers, and evidence-based finding verification. Use automatically whenever the user asks to review code, review local changes, inspect a PR, provides a PR number for review, or includes a GitHub pull-request URL with review intent, even without a slash command.
argument-hint: "[PR number]"
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(gh:*), Agent
---

# Review

Review the target supplied in `$ARGUMENTS` or identified in the user's natural-language request.

This plugin implements its own review workflow. Do not invoke external code
review plugins or treat their output as a prerequisite.

The review is read-only. Do not modify source files, install dependencies,
change repository configuration, or post GitHub comments unless the user
explicitly requests it.

Detailed review criteria are defined in `REVIEW.md`. Detailed responsibilities
and output schemas are defined by the agents under `agents/`.

## 1. Resolve the review target

Select one mode.

Recognize an explicit GitHub Pull Request URL anywhere in the user's request,
including `Review this PR: https://github.com/owner/repo/pull/123`. Treat
surrounding natural language as review instructions, not as part of the URL. If
multiple PR URLs or conflicting targets are present, reject the request as
ambiguous instead of guessing.

When the user invokes this Skill directly, accept either no arguments for
Developer mode or one numeric PR number for Reviewer mode. Do not accept a PR
URL as a direct Skill argument; ask the user to provide the URL as part of a
natural-language review request instead.

### Developer mode

When `$ARGUMENTS` is empty and the user's request does not identify a PR, review
commits ahead of the current
branch's upstream, staged changes, unstaged changes, and relevant untracked
source files.

Resolve the repository root, current and upstream branches, base and head SHAs,
changed files, additions, deletions, and complete diff. If no reviewable changes
exist, stop and report that there is nothing to review.

### Reviewer mode

When `$ARGUMENTS` contains one numeric PR number, or the user's natural-language
request contains a pull-request number or URL, resolve with `gh` the
repository, PR number, title, description, base and head branches and SHAs,
linked issues, changed files, additions, deletions, CI and check status, and
draft, closed, or merged state.

Reject ambiguous arguments instead of guessing. Do not alter the user's current
working tree. If code must be checked out, create an isolated temporary worktree
at the resolved head SHA and remove it after collecting the results.

Create one shared target context as an A2A-compatible Artifact named
`review.target`. Every agent must receive this same Artifact containing the
repository, base SHA, head SHA, diff, changed files, and PR metadata.

All inter-stage inputs and outputs must use the A2A-compatible Artifact defined
in `agents/README.md`. Validate the artifact name, media type, schema metadata,
and required payload fields, then pass the required Artifacts intact to the
next stage. Each receiving stage reads typed data from `parts[0].data`. Treat
missing or malformed artifact payloads as incomplete prerequisites; do not
reconstruct them from conversation history.

## 2. Check whether review is needed

In Reviewer mode, run `review:validation:review-needed` with the shared target
context and review metadata. It must check the conditions in this order: closed
or merged, draft, trivial, and already reviewed at the current head SHA by the
current authenticated reviewer.

If `should_review` is `false`, stop before context collection and report the
status and evidence concisely. Do not run `small-cls` or any review layer.

In Developer mode, skip this validation and continue when reviewable local
changes exist.

## 3. Collect and organize context

Run `review:context:context` with the shared target context, PR description,
related Issues, repository guidance, changed components, user-named sources,
and specification or decision references found in those discovery points.

Treat a related Issue as one discovery point rather than the preferred source
type. Do not hard-code or prefer a specific knowledge system. The context agent
must build bounded search anchors, prefer compatible MCP read-only tools when
available, search only relevant source families, and retrieve only the sections
needed to understand the change. For sources obtained through non-MCP tools,
each result must still use an MCP Resource-compatible `source.uri` plus a
precise `source.locator`.

Do not pass raw retrieved documents, search results, or the internal retrieval
plan to later agents. Preserve concise source-backed results, unknowns, and precise
locations in the collected context. The context agent must not classify facts
as requirements, create review questions, or assign review layers. If no
compatible retrieval tool is available, continue with available evidence and
record the resulting limitation.

## 4. Analyze Change Scope

Run `review:validation:small-cls` with the shared target context, PR metadata, changed
files, diff statistics, and resolved SHAs.

The agent must only analyze reviewer workload, Change Groups, scope
classification, and uncertainties. It must not decide whether review is needed
or produce code-quality findings.

If the result is `review_blocked`, continue only with checks that can still
produce reliable evidence. The final report must state that the review is
incomplete.

## 5. Build the review plan

As the orchestrator, read the repository's `REVIEW.md` and build the review
plan directly from the collected context, Change Scope result, PR
description, linked issues, changed files, and diff. At this stage, extract and
classify applicable requirements, acceptance criteria, constraints, and open
questions from the source-backed context. Assign stable review-only IDs and
preserve their source locations. Do not promote uncited context into a
normative requirement.

Consider all eight quality characteristics as a coverage check, but select only
the criteria relevant to this change. Use each criterion's applicability rules
to turn it into a concrete, PR-specific question. Assign every selected item to
one primary review layer:

- `structural`: design, dependencies, state, execution paths, performance,
  security, maintainability, and test design
- `contextual`: requirements, user value, PR intent, compatibility policy,
  migration decisions, and documentation

When another layer provides useful evidence, record it as a supporting layer.
Assign each selected item a stable identifier such as `RP-001`. Preserve the
review item ID, selected criterion, concrete question, selection reason,
primary layer, supporting layers, and expected evidence. Do not add generic
review items merely for completeness.

Package the completed review plan as an A2A-compatible Artifact named
`review.plan` with `metadata.schema: review/plan` before delegating review work.

Every structural and contextual review agent must return exactly one result
for every item assigned to it and preserve the item's `id`. Missing evidence must produce an
`insufficient_evidence` result rather than omission.

## 6. Run the review layers

After the review plan is complete, run these agents in parallel:

- `review:review:mechanical`
- `review:review:structural`
- `review:review:contextual`

Give the structural and contextual agents the shared target context, Change
Scope result, only the review items assigned to their `primary_layer`, relevant supporting-layer information,
and applicable repository guidance. Give the collected context to the contextual
reviewer; do not give it raw source documents or permission to expand the
retrieval scope.

Additionally, give CI and check status to the mechanical reviewer, the full
diff and codebase context to the structural reviewer, and PR descriptions,
issues, requirements, and documentation to the contextual reviewer.

Do not ask an agent to perform another layer's primary responsibility.

For each review delegation, explicitly include the repository root, review target,
base and head SHAs, changed files, complete diff or an unambiguous location for
it, assigned review items, and any agent-specific inputs required by its
definition. Do not assume that a subagent can recover orchestration state from
the parent conversation.

### Mechanical checks

The mechanical reviewer must discover and run the repository's existing
commands for applicable static analysis, lint, type checking, compilation,
Unit tests, and safe build or integration checks. Prefer commands used by CI.

Do not install dependencies, add tools, change configuration, or execute
destructive commands. For an external or otherwise untrusted pull request, do
not execute repository-controlled code without explicit user approval. Record
an A2A task failure when required verification cannot be started.

The mechanical Artifact uses `name: review.mechanical`. Its payload contains
one overall `result`, one `summary`, and a `checks` array containing only commands
that were actually executed. Each check records its name, command, `result`,
exit code, and observed summary. Results are only `passed` or `failed`.

## 7. Verify the review results

After all review layers finish, run `review:comment:comment` with the shared target
collected context, Change Scope result, complete review plan, both review
results, the complete `review.mechanical` Artifact, and the repository's
`REVIEW.md`.

The verifier must not perform another general review. It verifies candidate
findings against actual code, validates realistic failure paths and evidence,
rejects speculation and unrelated pre-existing issues, removes duplicates,
corrects classifications, and confirms whether applicable static analysis and
Unit tests ran.

Only the comment agent's `verified_results` may be passed to the final report.
Rejected results must not be presented as active findings. If required checks
did not run, preserve the reason and mark the review as incomplete.

The verifier must account for every `id` from the review plan. An
item may be represented by a verified result, a rejected result, or an explicit
incomplete reason, but it must not disappear silently.

## 8. Produce the final report

As the orchestrator, produce the final report using only the Change Scope
result, review plan, the comment agent's `verified_results`, and
`review_prerequisites`. Do not discover, add, remove, or re-evaluate
findings during formatting.

Return exactly these sections:

1. `Review Summary`
2. `Change Scope`
3. `Needs Your Attention`
4. `Review Coverage`

Do not expose internal agent names, processing layers, intermediate YAML,
rejected candidates, or orchestration details. Do not declare LGTM, Approve,
or Changes Requested. The final decision belongs to the human reviewer.

In `Review Summary`, show counts for potential problems, human decisions,
verified concerns, and items that could not be verified. Mark the review as
incomplete when a prerequisite is missing or a required check did not run
without a justified limitation.

In `Change Scope`, show changed files, additions, deletions, total lines, and
Change Groups. Explain a split recommendation only when applicable.

In `Needs Your Attention`, include only potential problems, human decisions,
and insufficient evidence. For potential problems, include the conclusion,
realistic failure scenario, review criterion, strongest evidence, and proposed
author comment.

In `Review Coverage`, group selected criteria by quality characteristic and
show the subcharacteristic, criterion, concrete result, and evidence. Do not
describe a verified result as an absolute safety guarantee.

## Completion requirements

Present the review as complete only when the target was resolved unambiguously,
Reviewer mode eligibility was confirmed,
required context was collected or its limitations were recorded, Change Scope
was evaluated, the review plan was generated from `REVIEW.md`, all applicable
review layers completed, applicable static analysis and Unit tests ran or have
justified limitations, candidate findings were independently verified, and the
final report contains only verified results.

If any requirement is missing, clearly mark the review as incomplete and state
the reason.
