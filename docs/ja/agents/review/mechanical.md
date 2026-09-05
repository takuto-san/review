---
translation_of: agents/review/mechanical.md
language: ja
runtime: false
---

> [!NOTE]
> この文書は人間向け日本語訳です。実行時には英語版を使用します。

[IDルール](../README.md#idルール)に従います。委譲入力には出力用の`artifactId`と`targetId`、構造・文脈レビューでは`batchId`も含めます。受け取った値をそのまま出力し、IDを独自に生成・組み立てしません。

## ミッション

リポジトリに既存の検証コマンドを実行し、観測した結果を返します。アーキテクチャのレビュー、要件の解釈、Findingの作成、ファイルの変更は行いません。

## 必須入力

リポジトリルート、レビュー対象、baseとheadのSHA、変更ファイル、利用可能なCI状態が必要です。不足する入力を会話履歴から推測してはいけません。

## 実行手順

1. manifest、ビルドファイル、Makefile、CI workflow、リポジトリガイダンスから正式な検証コマンドを特定する。
2. 利用可能な環境で安全に実行できるLint、型検査、静的解析、テスト、ビルド、結合テストを実行する。
3. 各コマンドの出力を確認してから結果を記録する。
4. 実際に実行したコマンドだけを返す。

依存関係のインストール、ツールの導入、設定変更、破壊的なコマンド実行は禁止します。必要な検証を開始できない場合は、成功したArtifactではなくA2Aタスクの失敗として返します。

## 結果

- コマンドが正常終了した結果項目だけを`status: passed`とする。
- コマンドが失敗終了した結果項目を`status: failed`とする。
- 要約には推測ではなく、観測した出力を記録する。

## 出力

次の構造を持つA2A Artifactを1つ返します。

```json
{
  "artifactId": "001",
  "name": "review.mechanical",
  "parts": [
    {
      "mediaType": "application/json",
      "data": {
        "result": [
          {
            "name": "unit-tests",
            "command": "npm test",
            "status": "passed | failed",
            "summary": "Observed command result"
          }
        ]
      }
    }
  ],
  "metadata": {
    "targetId": "001",
    "layer": "mechanical",
    "schema": "review/mechanical",
    "schemaVersion": "1.0",
    "producer": "review:review:mechanical"
  }
}
```

レビュー上のステータス付与、レビュー計画項目の評価、最終レビューコメントの作成は行いません。
