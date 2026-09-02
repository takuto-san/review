---
name: maintain-review-plugin
description: Keep the review plugin's agents, orchestration contracts, manifest, and Japanese and Simplified Chinese documentation synchronized. Use automatically whenever files under agents/, skills/, or .claude-plugin/ are added, removed, renamed, or behaviorally changed.
allowed-tools: Read, Glob, Grep, Edit, Write, Bash(git:*)
---

# Maintain Review Plugin

Maintain the plugin without changing its review policy beyond the user's request.

## Source of truth

- Runtime definitions under `agents/` and `skills/` are the English source of truth.
- Files under `docs/ja/` and `docs/zh-CN/` are human-readable translations and must retain `runtime: false`.
- Root `README.md`, `README.ja.md`, and `README.zh-CN.md` must describe the same user-visible behavior.
- `.claude-plugin/plugin.json` must register every runtime agent required by the review workflow.

## Required maintenance

When a runtime agent, skill, workflow contract, or manifest entry changes:

1. Identify every affected English runtime file and its Japanese and Simplified Chinese counterpart.
2. Preserve existing responsibilities and policy unless the user explicitly requests a behavioral change.
3. Synchronize headings, required inputs, boundaries, completion criteria, identifiers, and output fields across translations.
4. Update the three root READMEs when installation, usage, architecture, behavior, or customization guidance changes.
5. Check that every agent referenced by a skill exists and is registered in the plugin manifest.
6. Check that review-plan `review_item_id` values remain traceable through layer results and final verification.

Translate meaning naturally; do not mechanically translate code identifiers, paths, enum values, YAML keys, commands, or agent names.

## Completion checks

- Review `git diff` and preserve unrelated user changes.
- Run `git diff --check`.
- Parse `.claude-plugin/plugin.json` and confirm every registered path exists.
- Parse runtime agent frontmatter and confirm `name` and `description` are present.
- If the Claude CLI is available, run `claude plugin validate . --strict`; otherwise report that limitation.
- Do not declare completion while an affected Japanese or Simplified Chinese document remains stale.
