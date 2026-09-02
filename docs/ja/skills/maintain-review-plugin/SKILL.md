---
translation_of: skills/maintain-review-plugin/SKILL.md
language: ja
runtime: false
---

> [!NOTE]
> この文書は人間向け日本語訳です。実行時には英語版を使用します。

# Review Pluginの保守

`agents/`、`skills/`、`.claude-plugin/`の追加・削除・名称変更・振る舞い変更時に、runtime定義、manifest、日本語・簡体字中国語ドキュメントの整合性を維持します。

英語のruntime定義を正本とし、既存の責務と方針はユーザーが明示的に求めない限り変更しません。対応する翻訳では、見出し、必須入力、境界、完了条件、識別子、出力フィールドを同期します。利用方法や構成が変わる場合は3言語のルートREADMEも更新します。

完了前に、Skillが参照する全agentの存在とmanifest登録、`review_item_id`の追跡可能性、JSONとfrontmatterの構文、`git diff --check`を確認します。Claude CLIが利用可能なら`claude plugin validate . --strict`も実行します。コード識別子、パス、enum値、YAMLキー、コマンド、agent名は翻訳しません。
