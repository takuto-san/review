---
description: Review local changes or a GitHub pull request using specialized review agents, repository checks, and evidence-based verification
argument-hint: "[PR number or URL]"
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(gh:*), Agent
---

# Review

Review the target supplied in `$ARGUMENTS`.

This plugin implements its own review workflow. Do not invoke external code
review plugins or treat their output as a prerequisite.

The review is read-only. Do not modify source files, install dependencies,
change repository configuration, or post GitHub comments unless the user
explicitly requests it.

Detailed review criteria are defined in `REVIEW.md`. Detailed responsibilities
and output schemas are defined by the agents under `agents/`.

## 1. Resolve the review target

Select one mode.

### Developer mode

When `$ARGUMENTS` is empty, review commits ahead of the current branch's
upstream, staged changes, unstaged changes, and relevant untracked source files.

Resolve the repository root, current and upstream branches, base and head SHAs,
changed files, additions, deletions, and complete diff. If no reviewable changes
exist, stop and report that there is nothing to review.

### Reviewer mode

When `$ARGUMENTS` contains a pull-request number or URL, resolve with `gh` the
repository, PR number, title, description, base and head branches and SHAs,
linked issues, changed files, additions, deletions, CI and check status, and
draft, closed, or merged state.

Reject ambiguous arguments instead of guessing. Do not alter the user's current
working tree. If code must be checked out, create an isolated temporary worktree
at the resolved head SHA and remove it after collecting the results.

Create one shared target context. Every agent must receive the same repository,
base SHA, head SHA, diff, changed files, and PR metadata.

## 2. Analyze Change Scope

Run `change-scope-analyst` with the shared target context, PR metadata, changed
files, diff statistics, and resolved SHAs.

The agent must only analyze reviewability, Change Groups, scope classification,
and uncertainties. It must not produce code-quality findings.

If the result is `review_blocked`, continue only with checks that can still
produce reliable evidence. The final report must state that the review is
incomplete.

## 3. Build the review plan

Run `review-plan-builder` with the shared target context, Change Scope result,
PR description, linked issues, available requirements, changed files, diff,
and the repository's `REVIEW.md`.

The agent must consider all eight quality characteristics but select only the
criteria relevant to this change. Preserve every generated review item and its
`primary_layer` assignment. Do not add generic review items that were not
selected by the plan.

## 4. Run the review layers

After the review plan is complete, run these agents in parallel:

- `mechanical-reviewer`
- `structural-reviewer`
- `contextual-reviewer`

Give every agent the shared target context, Change Scope result, only the review
items assigned to its `primary_layer`, relevant supporting-layer information,
and applicable repository guidance.

Additionally, give CI and check status to `mechanical-reviewer`, the full diff
and codebase context to `structural-reviewer`, and PR descriptions, issues,
requirements, and documentation to `contextual-reviewer`.

Do not ask an agent to perform another layer's primary responsibility.

### Mechanical checks

The `mechanical-reviewer` must discover and run the repository's existing
commands for applicable static analysis, lint, type checking, compilation,
Unit tests, and safe build or integration checks. Prefer commands used by CI.

Do not install dependencies, add tools, change configuration, or execute
destructive commands. For an external or otherwise untrusted pull request, do
not execute repository-controlled code without explicit user approval. Record
blocked commands and their reasons as insufficient evidence.

## 5. Verify the review results

After all review layers finish, run `finding-verifier` with the shared target
context, Change Scope result, complete review plan, all three review results,
mechanical commands, and the repository's `REVIEW.md`.

The verifier must not perform another general review. It verifies candidate
findings against actual code, validates realistic failure paths and evidence,
rejects speculation and unrelated pre-existing issues, removes duplicates,
corrects classifications, and confirms whether applicable static analysis and
Unit tests ran.

Only `finding-verifier.verified_results` may be passed to the final report.
Rejected results must not be presented as active findings. If required checks
did not run, preserve the reason and mark the review as incomplete.

## 6. Produce the final report

Run `review-synthesizer` last with only the Change Scope result, review plan,
`finding-verifier.verified_results`, and
`finding-verifier.review_prerequisites`.

The synthesizer must format existing verified evidence. It must not discover,
add, remove, or re-evaluate findings.

Return exactly these sections:

1. `Review Summary`
2. `Change Scope`
3. `Needs Your Attention`
4. `Review Coverage`

Do not expose internal agent names, processing layers, intermediate YAML,
rejected candidates, or orchestration details. Do not declare LGTM, Approve,
or Changes Requested. The final decision belongs to the human reviewer.

## Completion requirements

Present the review as complete only when the target was resolved unambiguously,
Change Scope was evaluated, the review plan was generated from `REVIEW.md`, all
applicable review layers completed, applicable static analysis and Unit tests
ran or have justified limitations, candidate findings were independently
verified, and the final report contains only verified results.

If any requirement is missing, clearly mark the review as incomplete and state
the reason.
