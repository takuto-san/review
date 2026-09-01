---
description: Review local changes or a GitHub pull request with Claude Code, repository checks, and risk-aware reporting
argument-hint: "[PR number or URL]"
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(gh:*), Bash(npm:*), Bash(npx:*), Bash(pnpm:*), Bash(yarn:*), Bash(pytest:*), Bash(uv:*), Bash(poetry:*), Bash(mvn:*), Bash(gradle:*), Bash(./gradlew:*), Bash(make:*), Agent, Skill
---

# Review

Review the target supplied in `$ARGUMENTS`. The review is read-only: do not
modify code, install dependencies, or post GitHub comments unless the user
explicitly requests it.

## 1. Select the mode and target

- When `$ARGUMENTS` is empty, use **developer mode**. Review the current
  branch's commits ahead of its upstream together with uncommitted changes.
- When `$ARGUMENTS` contains a pull-request number or URL, use **reviewer
  mode**. Resolve its repository, base SHA, head SHA, title, description,
  linked issue, changed files, and CI status with `gh`.
- Reject ambiguous arguments instead of guessing the target.

In reviewer mode, do not replace or alter the user's current working tree.
Use the pull-request target supported by the review capability. If mechanical
checks require a checkout, use an isolated temporary worktree and remove it
after collecting the results.

## 2. Understand the change

Read the applicable `CLAUDE.md`, `REVIEW.md`, contribution guidance, build
files, package manifests, and CI workflows. Inspect the diff and surrounding
code.

Classify Change Scope:

- **Focused**: one self-contained change
- **Split recommended**: independent changes can reasonably be separated
- **Review blocked**: the scope or missing context prevents a reliable review

Summarize the purpose, affected components, externally visible behavior, and
important risks. Select only review concerns that apply to this change.

## 3. Run Claude Code review

Invoke Claude Code's official `code-review` capability for the selected
target. Pass the PR number or URL in reviewer mode; use the current diff in
developer mode.

Do not recreate, quote, or modify the official capability's implementation.
If it is unavailable, record the reason and continue with the checks that can
still run. A missing official review must be reported as unverified.

## 4. Run mechanical checks

Discover the repository's existing commands from CI workflows, manifests,
Makefiles, and repository guidance. Run all applicable existing commands for:

1. static analysis or lint
2. type checking
3. unit tests

Prefer the same commands used by CI. Record the exact command, exit status,
and concise result. Do not invent scripts, silently change configuration, or
claim that a command passed when it was not executed.

For an external or otherwise untrusted pull request, do not execute
repository-controlled code until the user explicitly approves it. Report the
blocked checks and why approval is required.

## 5. Decide whether an adversarial review is needed

Request an additional Codex adversarial review only when the change includes
one or more of the following:

- authentication, authorization, secrets, personal data, or payments
- destructive data operations, schema migrations, or rollback behavior
- concurrency, queues, retries, caching, or distributed state
- a breaking public API, protocol, event, or storage-format change
- a substantial responsibility or architecture change
- a realistic failure mode involving data loss or prolonged outage
- a disputed or low-confidence finding that benefits from independent
  challenge

When the Codex integration is installed, invoke its configured wrapper with
the target and a narrow focus derived from the risks above. Do not run a
second generic review. If Codex is unavailable, mark this optional check as
unavailable; do not fail the entire review.

## 6. Validate findings

Keep a finding only when all of the following are present:

- a concrete location in changed code
- a realistic trigger or execution path
- an observable impact
- evidence that the change introduced or exposed the problem

Remove duplicates, style preferences, generated-file findings, lockfile
findings, unrelated pre-existing issues, and errors already reported clearly
by mechanical checks.

Do not automatically turn an AI finding into a request to the author. Present
it to the human reviewer for confirmation.

## 7. Report

Output exactly these sections:

### Review Summary

State the target, purpose, overall result, and whether required checks ran.

### Change Scope

Show the scope classification and its evidence.

### Needs Your Attention

Include only:

- **Potential problem**
- **Human decision**
- **Could not verify**

For each potential problem, include the conclusion, trigger, impact, affected
review concern, code location, and what the reviewer should confirm.

### Review Coverage

Group applicable concerns from `REVIEW.md` and show:

| Concern | Result | Evidence |
|---|---|---|
| Applicable concern | Verified result or limitation | Code location, command, or source |

Finish with the commands executed and their pass, fail, blocked, or unavailable
status.
