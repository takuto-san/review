---
translation_of: agents/README.md
language: ja
runtime: false
---

# PRレビューエージェント

| エージェント | 責務 |
|---|---|
| `context` | 明示された参照から必要な情報だけを収集し、媒体非依存のコンテキストを作成する |
| `review-needed` | closed、draft、trivial、already-reviewedのPRをskipする |
| `small-cls` | 規模、Change Group、凝集性によるレビュワー負荷を判定する |
| `mechanical` | CI相当のテスト、静的解析、客観的な検証を実行する |
| `structural` | 設計、実行経路、状態、性能、セキュリティ、保守性を確認する |
| `contextual` | 要求、意図、互換性、文書を仕様駆動で確認する |
| `comment` | Findingを再検証し、推測と重複を除き、PRコメント候補を作る |

推奨順序はReviewer modeの`review-needed`、`context`、`small-cls`、`skills/review/SKILL.md`による計画作成、3層レビュー、`comment`、最終レポートです。3層レビューは並列実行できます。

各レビュー計画項目には一意な`review_item_id`を付け、レビューから検証まで維持します。各エージェントには必要な入力を明示的に渡し、レビューエージェントは割り当てられた項目ごとに必ず1件の結果を返します。証拠が不足する項目は省略せず`insufficient_evidence`とします。

機械的レビューは安全かつ適用可能な静的解析とUnitテストを実行し、全コマンドと結果を記録します。`comment`は未完了のレビューを完了扱いにしません。
