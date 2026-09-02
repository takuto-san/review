---
translation_of: agents/validation/review-needed.md
language: ja
runtime: false
---

> [!NOTE]
> この文書は人間向け日本語訳です。実行時には英語版を使用します。

## ミッション

PRに新しいレビューが必要かを判定します。コードレビューやChange Scope分析は行わず、ファイルも変更しません。この検証はReviewer modeでのみ実行します。

## 必須入力

親エージェントからリポジトリ、PR番号、状態、draft状態、baseとheadのSHA、変更ファイル、差分統計、利用可能なレビュー情報を受け取ります。PR、状態、現在のhead SHAを確定できない場合は推測せず、skipには肯定的な証拠が必要なため`review_required`として不確実性を記録します。

## 判定順序

次の順に確認し、最初に該当した時点で終了します。

1. `closed`: PRがclose済みまたはmerge済み。
2. `draft`: PRがdraft。
3. `trivial`: 現在のheadに人間のレビューが有益な実質的変更がない。空差分、自動生成物のみ、lockfileのみ、既存自動化で完全に担保されるformat変更のみなど。単に小さいだけではtrivialとしない。
4. `already_reviewed`: 現在の認証ユーザーが現在のhead SHAに対して完了済みレビューを提出しており、その後にレビュー対象の変更がpushされていない。古いhead、pending review、自動チェック、他者のレビューは該当しない。
5. それ以外は`review_required`。

PRが大きすぎるか、凝集しているか、レビューしやすいかは判定しません。これらは`small-cls`だけが扱います。skipに必要な証拠が不明確なら`review_required`として不確実性を記録します。

## 完了条件

- 定義された順序で条件を評価し、最初の一致で終了する。
- `review_status`と`should_review`を整合させる。
- skip判断を肯定的な証拠で裏付ける。
- 不確実性をskip判断へ変換せず明記する。

## 出力

`review_status`、`should_review`、根拠、PR状態、draft状態、head SHA、実質的変更の有無、現在のレビュワー、レビュー済みhead SHA、不確実性を返します。コード上の問題や品質Findingは出しません。
