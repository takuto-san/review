---
name: review
description: Review local code changes or a GitHub pull request. Use when the user asks to review code, inspect a PR, check a PR URL, or assess changes before commit or merge.
argument-hint: "[PR number or URL]"
disable-model-invocation: false
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(gh:*), Bash(npm:*), Bash(pnpm:*), Bash(yarn:*), Bash(pytest:*), Bash(make:*), Agent, Skill
---

# Review

Run a read-only review. Do not edit code or post comments unless explicitly requested.

## Modes

- No argument: developer mode; review commits ahead of upstream and uncommitted changes.
- PR number or URL: reviewer mode; review that pull request.

## Workflow

1. Ask `scope-and-plan-builder` to identify the target, scope, guidance, checks, and risks.
2. Invoke Claude Code's official `code-review` capability for that target.
3. Run existing static-analysis, type-checking, and unit-test commands discovered from CI, manifests, Makefiles, and repository guidance.
4. Do not invent commands or treat an unexecuted check as passing.
5. Ask `review-presenter` to consolidate all evidence.

Do not execute repository-controlled scripts from untrusted external PRs without explicit approval.
