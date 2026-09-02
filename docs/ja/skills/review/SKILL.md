---
translation_of: skills/review/SKILL.md
language: ja
runtime: false
---

> [!NOTE]
> この文書は人間向けの日本語訳です。実行時には英語版`skills/review/SKILL.md`を使用します。

# Review

ユーザーの自然言語から特定した対象をレビューします。「このPRをレビューして https://github.com/owner/repository/pull/123」のような依頼からClaudeが自動起動します。ユーザーが`/`コマンドとして直接実行することはできません。レビューは読み取り専用です。

## 1. レビュー対象の解決

自然言語のどこにPR番号またはURLが含まれていてもReviewer modeとして認識し、`gh`でPR、関連Issue、差分、CI状態を取得します。複数URLや競合する対象があれば推測しません。PRが指定されずローカル変更のレビューを依頼された場合はDeveloper modeを使用します。

## 2. コンテキストの収集と整理

`review:context:context`を実行します。関連Issueを優先入口としつつ特定媒体へ依存せず、明示された参照の必要箇所だけをEvidence Packetへ整理します。生文書は後続へ渡しません。取得不能、情報源の矛盾、正本区分、出典位置を保持します。

## 3. Change Scopeの分析

`review:validation:small-cls`を実行し、変更統計、Change Group、凝集性、レビュー可能性を判定します。`review_blocked`なら信頼できる確認だけを続行し、最終結果を未完了とします。

## 4. レビュー計画の作成

オーケストレーターが`REVIEW.md`の8品質特性を検討し、関係する観点だけをPR固有の問いへ変換して`mechanical`、`structural`、`contextual`へ割り当てます。選択理由、補助レイヤー、期待する証拠を保持します。

## 5. 3層レビューの実行

次を並列実行します。

- `review:review:mechanical`
- `review:review:structural`
- `review:review:contextual`

機械層にはCI情報、構造層には差分とコードベース、文脈層にはEvidence Packetを渡します。外部または信頼できないPRのコードは明示承認なしに実行しません。

## 6. レビュー結果の検証

`review:comment:comment`へEvidence Packet、Change Scope、計画、全結果、実行コマンド、`REVIEW.md`を渡します。失敗経路と証拠を再確認し、推測・既存問題・重複を除外します。検証済み結果だけを最終出力へ渡します。

## 7. 最終レポート

オーケストレーターがReview Summary、Change Scope、Needs Your Attention、Review Coverageだけを出力します。新しいFindingを作らず、内部エージェント名や中間YAMLを表示しません。最終判断は人間のレビュワーに委ねます。

対象、コンテキスト、Change Scope、計画、各レビュー層、必要な静的解析とUnitテスト、Finding検証のいずれかが不足する場合は、理由を示してレビュー未完了とします。
