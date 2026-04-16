# サブエージェント用プロンプトテンプレート

バッチモード Phase 2 で各 Agent を起動する際に、以下の構造でプロンプトを組み立てる。
`< >` 内の変数を実際の値に置き換えること。

テンプレート・スタイル規約の部分には、[wiki-paper-template.md](wiki-paper-template.md) の内容を**全文展開**して埋め込む。

---

```
あなたは ai-research-workspace の ingest-paper スキルのサブエージェントです。
以下の論文 PDF を読み、wiki ページを作成してください。

## 対象論文
- PDF ファイル: raw/papers/<filename>.pdf
- 作成する wiki ページ: wiki/papers/<name>.md
- bibkey: <bibkey>

## PDF の読み方
Read ツールで PDF を直接読む。pages パラメータで範囲を指定すること。
まず pages: "1-10" で読み、足りなければ続きを読む（1回あたり最大20ページ）。

## 本研究との関係
<current.md から抽出した 1〜2 文の要約>

## やること
1. PDF を Read ツールで読む（全文、必要な情報が揃うまで）
2. wiki/papers/<name>.md を作成（下記テンプレート・スタイル規約に従う）

以下はやらないこと（親エージェントが Phase 3 で一括処理する）:
- paper/common/refs.bib への追記
- wiki/index.md の更新
- log.md への追記
- wiki/concepts/ の更新

## テンプレート・スタイル規約
<wiki-paper-template.md の内容をここに展開>
```
