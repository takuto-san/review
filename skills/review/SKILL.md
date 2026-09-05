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

Follow the [ID rules](../../agents/README.md#id-rules) when assigning target, item, batch, and output Artifact IDs. Pass assigned output IDs explicitly to each agent.

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

In Reviewer mode, the orchestrator performs eligibility checking directly using
already collected metadata and the decision procedure and payload contract in
`skills/review/checks/eligibility.md`. Do not delegate a validation agent.
Preserve the condition order: closed or merged, draft, trivial, and already
reviewed at the current head SHA by the current authenticated reviewer.
Obtain missing facts through read-only commands. An uncertain skip condition
must result in continuing review with the uncertainty recorded. Produce
`review.eligibility` using the existing Artifact contract.

If `should_review` is `false`, stop before context collection and report the
status and evidence concisely. Do not collect context or run any review layer.

In Developer mode, skip this validation and continue when reviewable local
changes exist.

### Start mechanical checks early

Once eligibility passes (or reviewable local changes are found), start
`review:review:mechanical` concurrently with context collection. Provide the
repository root, target, base and head SHAs, changed files, CI status, and
assigned Artifact IDs. Apply the Mechanical checks safety rules below before
execution. Do not wait for context, scope analysis, or the review plan.
Retain the task handle and its result or failure; never launch it a second time
at the review-layer stage. Keep an isolated worktree available until all agents
using it finish, including mechanical checks.

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

As part of planning, the orchestrator analyzes scope directly using the
procedure and payload contract in `skills/review/checks/scope.md`. Do not
delegate a scope agent. Reuse collected metadata and diff statistics, group
substantive changes by purpose, account for all files, and assess cohesion and
reviewer workload. Produce `review.scope` before `review.plan`, preserving
uncertainties and the existing scope classifications. Scope analysis does not
produce code-quality findings or change eligibility.

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
Assign each selected item a stable identifier such as `001`. Preserve the
review item ID, selected criterion, concrete question, selection reason,
primary layer, supporting layers, and expected evidence. Do not add generic
review items merely for completeness.

Package the completed review plan as an A2A-compatible Artifact named
`review.plan` with `metadata.schema: review/plan` before delegating review work.

Every structural and contextual review agent must return exactly one result
for every item assigned to it and preserve the item's `id`. Each result contains
`assessment.evaluation`; missing evidence must produce
`assessment.evaluation.level: not_assessable` rather than omission.

## 6. Run the review layers

After the review plan is complete, run these agents in parallel while the
already-started mechanical checks continue:

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

Partition each structural and contextual layer's assigned items into batches of
at most five related items before delegation. Prefer three to five items when
available; allow smaller batches and never add irrelevant items to fill a batch.
Each invocation evaluates only its batch and returns one result per assigned ID.
Give every batch the required shared context and a target-local unique batch ID
such as `"001"`. Assign numeric-string Artifact IDs using the ID rules in `agents/README.md`.
Store `targetId`, `batchId`, and `layer` in metadata. The consolidated
Artifact receives a new `artifactId` and omits `batchId`. Before verification, consolidate
batch results into one Artifact per layer using the existing layer schema, and
check that every assigned ID appears exactly once with no missing or extra IDs.

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

The mechanical Artifact uses `name: review.mechanical`. Its payload contains a
`result` array with only commands that were actually executed. Each entry
records its name, command, `status`, and observed summary. Statuses
are only `passed` or `failed`.

## 7. Verify the review results

Wait for the early mechanical task and all structural/contextual batches to
finish, preserving failures as incomplete prerequisites. Then run `review:comment:comment` with the shared target
collected context, Change Scope result, complete review plan, the complete
`review.structural`, `review.contextual`, and `review.mechanical` Artifacts, and
the repository's `REVIEW.md`.

The verifier must not perform another general review. It verifies candidate
findings against actual code, validates realistic failure paths and evidence,
rejects speculation and unrelated pre-existing issues, removes duplicates,
assigns the final five-label classification from each layer's `assessment.evaluation` and
`assessment`, and confirms whether applicable static analysis and Unit tests ran.

Only the comment agent's `verified_results` may be passed to the final report.
Rejected results must not be presented as active findings. If required checks
did not run, preserve the reason and mark the review as incomplete.

The verifier must account for every `id` from the review plan. An
item may be represented by a verified result, a rejected result, or an explicit
incomplete reason, but it must not disappear silently.

## 8. Produce the final report

As the orchestrator, produce the final report using only the Change Scope
result, review plan, and the comment agent's structured verification payload
(including `verified_results`, `mechanical_results`, `label_counts`,
`overall_label`, and `review_prerequisites`). Do not discover, add, remove, or re-evaluate
findings during formatting.

State that the labels and suggested fixes are advisory triage candidates for human review; they do not automatically authorize merge, rejection, or author requests.

Present one consolidated table with the columns `Review Layer`, `Review Item`,
`Label`, and `Result / Evidence`. Include every executed Mechanical check and
every Structural and Contextual review-plan item. Use only `Nit`, `LGTM`,
`Please Fix`, `Need Review`, or `Unable to Verify` as labels. After the table,
show counts for all five labels, including zero counts, and the overall label
calculated by the verifier. Preserve concrete evidence and missing-information
details, but do not expose intermediate Artifact data or rejected candidates.

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
