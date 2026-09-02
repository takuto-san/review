# PR Agent agents

| Agent | Responsibility |
|---|---|
| `change-scope-analyst` | PRの規模、Change Group、レビュー可能性を判定する |
| `review-plan-builder` | `REVIEW.md`から適用する品質特性・副特性・レビュー観点を選ぶ |
| `mechanical-reviewer` | CI、テスト、静的解析など機械的な項目を確認する |
| `structural-reviewer` | 設計、実行経路、状態、性能、セキュリティなどを確認する |
| `contextual-reviewer` | 要件、意図、ユーザー価値、互換性方針、文書を確認する |
| `comment` | Finding候補を再検証し、重複・推測・誤分類を除去してPRコメント候補を作る |
| `review-synthesizer` | 検証済み結果を4セクションの最終表示へ変換する |

推奨実行順序:

1. `change-scope-analyst`
2. `review-plan-builder`
3. `mechanical-reviewer`、`structural-reviewer`、`contextual-reviewer`
4. `comment`
5. `review-synthesizer`

3つのレビューエージェントは、`review-plan-builder`が割り当てた項目を並列に評価できます。

## 必須条件

- `mechanical-reviewer`は、リポジトリで定義されている静的解析とUnitテストを実際に実行する。
- 実行した検証コマンドと結果を記録する。
- `comment`は各レビュー層と機械的チェックの完了条件を確認し、満たされていないレビューを完了扱いにしない。
