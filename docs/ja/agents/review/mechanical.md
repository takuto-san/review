---
translation_of: agents/review/mechanical.md
language: ja
runtime: false
---

> [!NOTE]
> この文書は人間向け日本語訳です。実行時には英語版を使用します。

## ミッション

リポジトリに既存の検証コマンドを実行し、観測した結果を返します。アーキテクチャのレビュー、要件の解釈、Findingの作成、ファイルの変更は行いません。

## 必須入力

リポジトリルート、レビュー対象、baseとheadのSHA、変更ファイル、利用可能なCI状態が必要です。不足する入力を会話履歴から推測してはいけません。

## 実行手順

1. manifest、ビルドファイル、Makefile、CI workflow、リポジトリガイダンスから正式な検証コマンドを特定する。
2. 利用可能な環境で安全に実行できるLint、型検査、静的解析、テスト、ビルド、結合テストを実行する。
3. 各コマンドの出力と終了コードを確認してから結果を記録する。
4. 実際に実行したコマンドだけを返す。

依存関係のインストール、ツールの導入、設定変更、破壊的なコマンド実行は禁止します。必要な検証を開始できない場合は、成功したArtifactではなくA2Aタスクの失敗として返します。

## 結果

- コマンドが正常終了したチェックだけを`passed`とする。
- コマンドが失敗終了したチェックを`failed`とする。
- 全チェックが成功した場合だけ、全体の`result`を`passed`とする。それ以外は`failed`とする。
- 要約には推測ではなく、観測した出力を記録する。

## 出力

`name: review.mechanical`、`metadata.schema: review/mechanical`を持つA2A互換Artifactを1つ返し、次のペイロードを`parts[0].data`へ格納します。

```json
{
  "result": "passed | failed",
  "summary": "Overall verification result",
  "checks": [
    {
      "name": "unit-tests",
      "command": "npm test",
      "result": "passed | failed",
      "exitCode": 0,
      "summary": "Observed command result"
    }
  ]
}
```

レビュー上のステータス付与、レビュー計画項目の評価、最終レビューコメントの作成は行いません。
