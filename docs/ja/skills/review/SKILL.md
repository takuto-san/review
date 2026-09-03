---
translation_of: skills/review/SKILL.md
language: ja
runtime: false
---

> [!NOTE]
> この文書は人間向けの日本語訳です。実行時には英語版`skills/review/SKILL.md`を使用します。

# Review

`$ARGUMENTS`または自然言語から対象を特定します。`/review:review`はローカル変更、`/review:review 123`はPR番号をレビューします。PR URLは直接引数にせず、「このPRをレビューして URL」のような自然言語で指定します。レビューは読み取り専用です。

工程間の入力と出力はすべて`agents/README.md`で定義するA2A互換Artifactを使用します。オーケストレーターはArtifact名、media type、schema metadata、必須ペイロードを検証し、Artifactを保持したまま次工程へ渡します。受信側は`parts[0].data`を読み取り、不足データを会話履歴から復元しません。共有対象は`review.target`、レビュー計画は`review.plan`として表現します。

## 1. レビュー対象の解決

自然言語のどこにPR番号またはURLが含まれていてもReviewer modeとして認識し、`gh`でPR、関連Issue、差分、CI状態を取得します。複数URLや競合する対象があれば推測しません。PRが指定されずローカル変更のレビューを依頼された場合はDeveloper modeを使用します。

## 2. レビュー要否の判定

Reviewer modeでは`review:validation:review-needed`を実行し、closedまたはmerged、draft、trivial、現在のhead SHAを現在の認証ユーザーがalready-reviewedの順に確認します。`should_review`がfalseなら、根拠を簡潔に報告して終了し、`small-cls`やレビュー層は実行しません。Developer modeではこの判定を行いません。

## 3. コンテキストの収集と整理

`review:context:context`を実行します。関連Issueを特権的な入口にせず、ユーザー指定の情報源、PR関連成果物、変更に隣接するリポジトリ情報から境界付き検索アンカーを作ります。利用可能な場合はMCP読み取りツールを優先し、それ以外で取得した情報源も各resultのMCP Resource互換`source.uri`と正確な`source.locator`で識別します。生文書や内部取得計画は後続へ渡さず、変更目的、出典付きの`results`、`unknowns`だけを保持します。Requirement分類やレビュー質問作成は行いません。

## 4. Change Scopeの分析

`review:validation:small-cls`を実行し、変更統計、Change Group、凝集性、レビュー可能性を判定します。`review_blocked`なら信頼できる確認だけを続行し、最終結果を未完了とします。

`small-cls`はレビュー要否ではなく、変更量とChange Groupがレビュワーに過大な負荷を与えないかだけを判定します。

## 5. レビュー計画の作成

オーケストレーターが収集済みコンテキストから適用可能なRequirement、Acceptance Criterion、制約、未決事項を抽出・分類し、出典位置を保ったレビュー用IDを付けます。出典のない情報を正式Requirementへ昇格させません。そのうえで`REVIEW.md`の8品質特性を検討し、関係する観点だけをPR固有の問いへ変換して`structural`または`contextual`へ割り当てます。各項目へ`RP-001`のような安定した`id`を付け、選択理由、補助レイヤー、期待する証拠とともに保持します。構造・文脈レビューエージェントは割り当てられた項目ごとに必ず1件の結果を返し、証拠不足を省略せず`insufficient_evidence`とします。

## 6. 3層レビューの実行

次を並列実行します。

- `review:review:mechanical`
- `review:review:structural`
- `review:review:contextual`

機械層にはCI情報を渡し、既存の検証コマンドを実行させます。`review.mechanical` Artifactは、全体の`result`と`summary`、実行したコマンドだけを含む`checks`を返します。各チェックは名前、コマンド、`result`、終了コード、観測した要約を持ち、結果は`passed`または`failed`だけです。構造層には差分とコードベース、文脈層には収集済みコンテキストを渡します。外部または信頼できないPRのコードは明示承認なしに実行しません。

各委譲では、リポジトリルート、レビュー対象、baseとheadのSHA、変更ファイル、完全な差分またはその明確な位置、割り当て項目、agent固有の必須入力を明示します。subagentが親会話からオーケストレーション状態を復元できるとは仮定しません。

## 7. レビュー結果の検証

`review:comment:comment`へ収集済みコンテキスト、Change Scope、計画、構造・文脈レビュー結果、完全な`review.mechanical` Artifact、`REVIEW.md`を渡します。失敗経路と証拠を再確認し、推測・既存問題・重複を除外します。検証済み結果だけを最終出力へ渡します。計画内の全`id`を検証済み結果、除外結果、明示的な未完了理由のいずれかで処理し、項目を黙って消失させません。

## 8. 最終レポート

オーケストレーターがReview Summary、Change Scope、Needs Your Attention、Review Coverageだけを出力します。新しいFindingを作らず、内部エージェント名や中間YAMLを表示しません。最終判断は人間のレビュワーに委ねます。

対象、コンテキスト、Change Scope、計画、各レビュー層、必要な静的解析とUnitテスト、Finding検証のいずれかが不足する場合は、理由を示してレビュー未完了とします。
