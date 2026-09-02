---
translation_of: agents/validation/review-needed.md
language: ja
runtime: false
---

> [!NOTE]
> この文書は人間向け日本語訳です。実行時には英語版を使用します。

PRに新しいレビューが必要かを判定します。コードレビューやChange Scope分析は行わず、ファイルも変更しません。この検証はReviewer modeでのみ実行します。

## 判定順序

次の順に確認し、最初に該当した時点で終了します。

1. `closed`: PRがclose済みまたはmerge済み。
2. `draft`: PRがdraft。
3. `trivial`: 現在のheadに人間のレビューが有益な実質的変更がない。空差分、自動生成物のみ、lockfileのみ、既存自動化で完全に担保されるformat変更のみなど。単に小さいだけではtrivialとしない。
4. `already_reviewed`: 現在の認証ユーザーが現在のhead SHAに対して完了済みレビューを提出しており、その後にレビュー対象の変更がpushされていない。古いhead、pending review、自動チェック、他者のレビューは該当しない。
5. それ以外は`review_required`。

PRが大きすぎるか、凝集しているか、レビューしやすいかは判定しません。これらは`small-cls`だけが扱います。skipに必要な証拠が不明確なら`review_required`として不確実性を記録します。

## 出力

`review_status`、`should_review`、根拠、PR状態、draft状態、head SHA、実質的変更の有無、現在のレビュワー、レビュー済みhead SHA、不確実性を返します。コード上の問題や品質Findingは出しません。
