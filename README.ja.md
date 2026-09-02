# review

[English](README.md) | 日本語 | [简体中文](README.zh-CN.md)

ローカル変更およびGitHub Pull RequestをレビューするためのClaude Codeプラグインです。

一般的な多段階レビューの考え方を参考にしたクリーンルーム実装です。コンテキスト収集、レビュー計画、専門化されたレビュー層、Findingの検証、最終レポート生成は、このプラグイン内で完結します。

## ドキュメント

- [日本語ドキュメント](docs/ja/README.md)
- [英語ドキュメント](README.md)
- [简体中文文档](docs/zh-CN/README.md)

Claude Codeのエージェントは、`skills/`、`agents/`、およびルートの`REVIEW.md`にある英語ファイルを読み込みます。`docs/ja/`と`docs/zh-CN/`は人間向けの翻訳であり、レビューエージェントには読み込まれません。

## 処理フロー

### Developer mode

現在のローカルリポジトリにあるコミットと作業ツリーの変更をレビューします。

```mermaid
flowchart TD
    A[ローカル変更を特定する<br/>upstreamより先のコミット・ステージ済み・未ステージ・関連する未追跡ファイル] --> B[ローカル変更の意図を理解する<br/>コミットメッセージ・リポジトリガイド・明示された参照を確認]
    B --> C[必要な背景情報だけを集める<br/>ローカル変更から参照できる資料だけを確認]
    C --> D[レビュー用の情報に整理する<br/>要求・制約・未決事項・出典]
    D --> E[レビュー可能な変更か確認する<br/>規模・目的のまとまり・分割可能性]
    E --> F[今回確認すべき項目を決める<br/>関係する品質観点だけを選択]

    F --> G1[客観的なチェックを実行する<br/>CI・静的解析・型・テスト]
    F --> G2[コードの構造を確認する<br/>ロジック・依存関係・状態・失敗経路]
    F --> G3[実装と目的を照合する<br/>要求・受け入れ条件・制約]

    G1 --> H[レビュー結果を検証する<br/>根拠確認・重複排除・推測の除外]
    G2 --> H
    G3 --> H
    H --> I[レビュー結果を提示する<br/>要約・変更範囲・要確認事項・確認範囲]
```

### Reviewer mode

PR番号またはURLで指定されたGitHub Pull Requestをレビューします。

```mermaid
flowchart TD
    A[Pull Requestを特定する<br/>タイトル・説明・ブランチ・差分・CI状態] --> B[依頼された変更を理解する<br/>PRと関連Issueを確認]
    B --> C[必要な背景情報だけを集める<br/>明示された仕様・判断資料だけを参照]
    C --> D[レビュー用の情報に整理する<br/>要求・制約・未決事項・出典]
    D --> E[PRがレビュー可能か確認する<br/>規模・目的のまとまり・分割可能性]
    E --> F[今回確認すべき項目を決める<br/>関係する品質観点だけを選択]

    F --> G1[客観的なチェックを実行する<br/>CI・静的解析・型・テスト]
    F --> G2[コードの構造を確認する<br/>ロジック・依存関係・状態・失敗経路]
    F --> G3[実装と目的を照合する<br/>要求・受け入れ条件・制約]

    G1 --> H[レビュー結果を検証する<br/>根拠確認・重複排除・推測の除外]
    G2 --> H
    G3 --> H
    H --> I[レビュー結果を提示する<br/>要約・変更範囲・要確認事項・確認範囲]
```

`context`エージェントは、利用者の環境で使用可能な任意の読み取り専用情報源を利用できます。レビュー対象から明示的に参照された情報だけをたどり、生の文書ではなく、必要な情報を整理したEvidence Packetをレビュー層へ渡します。

## 構成

```text
review/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── context/
│   │   └── context.md
│   ├── validation/
│   │   └── small-cls.md
│   ├── review/
│   │   ├── mechanical.md
│   │   ├── structural.md
│   │   └── contextual.md
│   ├── comment/
│   │   └── comment.md
├── skills/
│   └── review/
│       └── SKILL.md
├── REVIEW.md
├── docs/
│   ├── ja/
│   └── zh-CN/
├── README.md
├── README.ja.md
├── README.zh-CN.md
└── LICENSE
```

## 使用方法

```text
/review:review
/review:review 123
/review:review https://github.com/owner/repository/pull/123
ローカル変更をレビューして
このPRをレビューして https://github.com/owner/repository/pull/123
```

`/review:review`による明示実行だけでなく、自然言語のレビュー依頼からもSkillが自動選択されます。依頼のどこかにPR番号またはURLがあればReviewer mode、PRを指定せず現在またはローカル変更のレビューを依頼した場合はDeveloper modeになります。

レビュー開始前に、`context`エージェントがレビュー対象から明示的に参照された情報だけを取得し、媒体に依存しない簡潔なEvidence Packetへ変換します。Notion、Confluence、Google Docs、GitHub、Web、リポジトリ内文書などの生データを、そのままレビューエージェントへ渡すことはありません。

## 開発

```bash
claude plugin validate . --strict
claude --plugin-dir .
```

MIT Licenseで提供されます。
