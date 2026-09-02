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

推奨順序：

1. Reviewer modeでは`review-needed`
2. `context`
3. `small-cls`
4. `skills/review/SKILL.md`がレビュー計画を作成
5. `mechanical`、`structural`、`contextual`
6. `comment`
7. `skills/review/SKILL.md`が最終レポートを作成

3つのレビューエージェントは、割り当て項目を並列に評価できます。

## 完了条件

- 各レビュー計画項目には安定した`review_item_id`を付け、レビューから検証まで維持する。
- 各エージェントには必須入力を明示的に渡し、親会話から編成状態を推測させない。
- 各レビューエージェントは割り当て項目ごとに必ず1件の結果を返し、省略せず`insufficient_evidence`を使用する。
- `mechanical`は安全かつ適用可能な場合、リポジトリ定義の静的解析と単元テストを実行する。
- 実行したすべての検証コマンドと結果を記録する。
- `comment`は各レイヤーとチェックの完了を検証し、未完了レビューを完了扱いにしない。
