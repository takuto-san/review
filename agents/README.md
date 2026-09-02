# PR Agent agents

| Agent | Responsibility |
|---|---|
| `context` | 明示された参照から必要な情報だけを収集し、媒体非依存のEvidence Packetを作成する |
| `small-cls` | PRの規模、Change Group、凝集性、レビュー可能性を判定する |
| `mechanical-reviewer` | CI、テスト、静的解析など機械的な項目を確認する |
| `structural-reviewer` | 設計、実行経路、状態、性能、セキュリティなどを確認する |
| `contextual-reviewer` | 要件、意図、ユーザー価値、互換性方針、文書を確認する |
| `comment` | Finding候補を再検証し、重複・推測・誤分類を除去してPRコメント候補を作る |

推奨実行順序:

1. `context`
2. `small-cls`
3. `commands/review.md`がレビュー計画を作成する
4. `mechanical-reviewer`、`structural-reviewer`、`contextual-reviewer`
5. `comment`
6. `commands/review.md`が最終レポートを生成する

3つのレビューエージェントは、`commands/review.md`が割り当てた項目を並列に評価できます。

## 必須条件

- `mechanical-reviewer`は、リポジトリで定義されている静的解析とUnitテストを実際に実行する。
- 実行した検証コマンドと結果を記録する。
- `comment`は各レビュー層と機械的チェックの完了条件を確認し、満たされていないレビューを完了扱いにしない。
