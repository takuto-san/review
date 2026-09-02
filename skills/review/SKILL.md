---
name: review
description: Review local code changes or a GitHub Pull Request using context collection, scope validation, three specialized review layers, and evidence-based finding verification. Use automatically whenever the user asks to review code, review local changes, inspect a PR, provides a PR number for review, or includes a GitHub pull-request URL with review intent, even without a slash command.
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

Create one shared target context. Every agent must receive the same repository,
base SHA, head SHA, diff, changed files, and PR metadata.

## 2. Collect and organize context

Run `review:context:context` with the shared target context, PR description,
related Issues, repository guidance, changed components, and every explicit
specification or decision reference found in those sources.

Treat a related Issue as the preferred reference index when one exists. Do not
hard-code or prefer a specific knowledge system. The context agent must use
whatever compatible read-only tools are available, follow only explicit
references, retrieve only the sections needed to answer review questions, and
return a compact, source-independent `evidence_packet`.

Do not pass raw retrieved documents or search results to later agents. Preserve
unresolved references, conflicting sources, source authority, and precise
locations in the Evidence Packet. If no compatible retrieval tool is available,
continue with available evidence and record the resulting limitation.

## 3. Analyze Change Scope

Run `review:validation:small-cls` with the shared target context, PR metadata, changed
files, diff statistics, and resolved SHAs.

The agent must only analyze reviewability, Change Groups, scope classification,
and uncertainties. It must not produce code-quality findings.

If the result is `review_blocked`, continue only with checks that can still
produce reliable evidence. The final report must state that the review is
incomplete.

## 4. Build the review plan

As the orchestrator, read the repository's `REVIEW.md` and build the review
plan directly from the shared target context, Evidence Packet, Change Scope
result, PR description, linked issues, available requirements, changed files,
and diff.

Consider all eight quality characteristics as a coverage check, but select only
the criteria relevant to this change. Use each criterion's applicability rules
to turn it into a concrete, PR-specific question. Assign every selected item to
one primary layer:

- `mechanical`: CI, static analysis, tests, and objective repository checks
- `structural`: design, dependencies, state, execution paths, performance,
  security, maintainability, and test design
- `contextual`: requirements, user value, PR intent, compatibility policy,
  migration decisions, and documentation

When another layer provides useful evidence, record it as a supporting layer.
Preserve the selected criterion, concrete question, selection reason, primary
layer, supporting layers, and expected evidence. Do not add generic review
items merely for completeness.

## 5. Run the review layers

After the review plan is complete, run these agents in parallel:

- `review:review:mechanical`
- `review:review:structural`
- `review:review:contextual`

Give every agent the shared target context, Change Scope result, only the review
items assigned to its `primary_layer`, relevant supporting-layer information,
and applicable repository guidance. Give the Evidence Packet to the contextual
reviewer; do not give it raw source documents or permission to expand the
retrieval scope.

Additionally, give CI and check status to the mechanical reviewer, the full
diff and codebase context to the structural reviewer, and PR descriptions,
issues, requirements, and documentation to the contextual reviewer.

Do not ask an agent to perform another layer's primary responsibility.

### Mechanical checks

The mechanical reviewer must discover and run the repository's existing
commands for applicable static analysis, lint, type checking, compilation,
Unit tests, and safe build or integration checks. Prefer commands used by CI.

Do not install dependencies, add tools, change configuration, or execute
destructive commands. For an external or otherwise untrusted pull request, do
not execute repository-controlled code without explicit user approval. Record
blocked commands and their reasons as insufficient evidence.

## 6. Verify the review results

After all review layers finish, run `review:comment:comment` with the shared target
context, Evidence Packet, Change Scope result, complete review plan, all three
review results, mechanical commands, and the repository's `REVIEW.md`.

The verifier must not perform another general review. It verifies candidate
findings against actual code, validates realistic failure paths and evidence,
rejects speculation and unrelated pre-existing issues, removes duplicates,
corrects classifications, and confirms whether applicable static analysis and
Unit tests ran.

Only the comment agent's `verified_results` may be passed to the final report.
Rejected results must not be presented as active findings. If required checks
did not run, preserve the reason and mark the review as incomplete.

## 7. Produce the final report

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
required context was collected or its limitations were recorded, Change Scope
was evaluated, the review plan was generated from `REVIEW.md`, all applicable
review layers completed, applicable static analysis and Unit tests ran or have
justified limitations, candidate findings were independently verified, and the
final report contains only verified results.

If any requirement is missing, clearly mark the review as incomplete and state
the reason.
