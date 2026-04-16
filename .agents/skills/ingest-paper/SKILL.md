---
name: ingest-paper
description: >
  raw/papers/ にある論文 PDF を読み込み、wiki/papers/ に要約ページを作成し、
  wiki/concepts/ を更新し、paper/common/refs.bib に BibTeX を追加し、log.md に履歴を追記するスキル。
  論文の取り込み、PDF の読み込み、文献整理、サーベイ、wiki への追加、論文要約ページの作成など、
  論文を知識ベースに組み込む作業全般で使う。
  「この論文を読んで」「PDF を wiki に追加して」「まだ取り込んでない論文を処理して」
  のような依頼が来たら必ずこのスキルを使う。
disable-model-invocation: true
argument-hint: /ingest-paper <pdf-filename | "all">
---

# ingest-paper スキル

論文 PDF を読み、wiki 知識ベースに取り込むスキル。**単体モード**と**バッチモード**がある。

---

## 前提

- 対象 PDF が `raw/papers/` に置かれていること
- `wiki/index.md` / `wiki/papers/` / `wiki/concepts/` が存在すること
- Claude Code の Read ツールで PDF を直接読む（外部ツール不要）

## モード判定

| 引数 | モード | 処理 |
|---|---|---|
| 特定のファイル名 | **単体モード** | その 1 本だけ処理 |
| `all`、「全て」、「まだ取り込んでいない全て」等 | **バッチモード** | 未 ingest の全 PDF を並列処理 |

---

## 参照する meta reference

処理前に以下を読む（バッチモードでは Phase 1 で 1 度だけ）:

- **`ai-guide/venues.md`** — venue の Tier 分類
- **`wiki/reference-strategy.md`** — 引用カテゴリと「即追加すべき論文」リスト
- **`my-research/current.md`** — 本研究との関連付け用

---

## PDF の読み方

Claude Code の Read ツールは PDF を直接読める。論文は通常 10 ページ前後なので、ページ範囲を指定して読む。

```
Read(file_path: "raw/papers/<filename>.pdf", pages: "1-10")
```

10 ページを超える論文は分割して読む（1 回あたり最大 20 ページ）。
要約ページの作成に必要な情報（タイトル、著者、Abstract、手法、評価、関連研究）が揃うまで読み進める。

---

## 命名規則

PDF ファイル名は `<第一著者姓小文字><年><短縮名>.pdf` に統一する。

例:
- `menendez2017alive-infer.pdf`
- `meng2013lase.pdf`
- `bader2019getafix.pdf`

**既に命名規則に合致しているファイルはスキップ。** 合致していない場合は PDF の冒頭を読んで第一著者姓・発表年・短縮名を特定し、`mv` でリネームする。以降は新しいファイル名を使う。

---

## 単体モード

引数で指定された 1 本の PDF を処理する。Agent は使わず直接実行する。

### 手順

1. **PDF 読み込み**: Read ツールで `raw/papers/<filename>.pdf` を読む
2. **リネーム判定**: ファイル名が命名規則に合致していなければリネーム
3. **meta reference の読み込み**: `ai-guide/venues.md`、`wiki/reference-strategy.md`、`my-research/current.md` を読む
4. **wiki ページ作成**: [references/wiki-paper-template.md](references/wiki-paper-template.md) を Read し、テンプレート・スタイル規約に従い `wiki/papers/<name>.md` を作成
5. **refs.bib 追記**: `paper/common/refs.bib` に BibTeX エントリを追加
6. **wiki/index.md 更新**: 適切なセクションにリンクを追加（既存エントリがあれば「未 ingest」→ リンクに更新）
7. **log.md 追記**: ingest 履歴を追記（フォーマットは後述）
8. **concepts 更新**: 関連する `wiki/concepts/` ページを追記・新規作成

---

## バッチモード (3 フェーズ)

### Phase 1: 準備 (逐次)

#### 1-1. 未 ingest PDF の特定

```bash
ls raw/papers/*.pdf
ls wiki/papers/*.md
```

`wiki/papers/` に対応する `.md` がない PDF が未 ingest。

#### 1-2. PDF リネーム

未 ingest の PDF を Read ツールで冒頭だけ読み（pages: "1"）、命名規則に合致していなければリネーム。

#### 1-3. meta reference の読み込み

`ai-guide/venues.md`、`wiki/reference-strategy.md`、`my-research/current.md` を読み、以下をメモする:
- 本研究の概要 (1 段落)
- 各 PDF が reference-strategy のどの優先度に該当するか

### Phase 2: 並列 ingest (Agent)

**Agent ツールで各論文を並列処理する。**

各 Agent の責務は **wiki/papers/<name>.md の作成のみ**:
1. PDF を Read ツールで全文読む
2. `wiki/papers/<name>.md` を作成（テンプレート + スタイル規約に従う）

Agent プロンプトの組み立て方は [references/agent-prompt-template.md](references/agent-prompt-template.md) を参照。

**Agent は `run_in_background: true` で起動し、全て並列実行する。**

共有ファイル（refs.bib / wiki/index.md / log.md / wiki/concepts/）への書き込みは Agent にやらせない。複数 Agent が同時に同じファイルを編集すると競合するため、Phase 3 で一括処理する。

### Phase 3: 統合 (逐次、全 Agent 完了後)

#### 3-1. 成果物の検証

```bash
ls wiki/papers/          # 全ページが揃っているか
```

不足があれば Agent の出力を確認し、手動で補完する。

#### 3-2. refs.bib 一括追記

各 wiki/papers/<name>.md の frontmatter (title, authors, venue, year, bibkey) を読み取り、`paper/common/refs.bib` に BibTeX エントリを一括追加する。

```bash
grep '^@' paper/common/refs.bib  # 既存エントリとの重複確認
```

#### 3-3. wiki/index.md 一括更新

各論文を適切なセクション（本論文で直接使う / 将来使える / 汎用知識）に分類してリンクを追加。分類は reference-strategy.md と my-research/current.md を基に判断する。

#### 3-4. log.md 一括追記

全論文の ingest 履歴をまとめて追記する。

#### 3-5. concepts の統合更新

各 wiki/papers/*.md の内容を踏まえて:
- 関連する既存 concepts ページを追記
- 新規 concept が必要な場合は作成

#### 3-6. ユーザーに報告

作成・更新したファイル一覧と、reference-strategy.md に照らして次に読むべき候補論文を提示。

---

## log.md のフォーマット

```markdown
## [YYYY-MM-DD] ingest | <論文タイトル略称> (<venue> <year>)
- paper: wiki/papers/<name>.md
- concepts updated: <更新した concept のリスト、なければ (なし)>
- concepts created: <新規作成した concept のリスト、なければ (なし)>
- notes: <本研究 (my-research/current.md) との関係を 1〜3 文で。どの §章に影響するかも明記>
```

バッチモードでは各論文ごとに 1 エントリ書く。

---

## references

| ファイル | 内容 | 使うタイミング |
|---|---|---|
| [references/wiki-paper-template.md](references/wiki-paper-template.md) | wiki/papers テンプレート + 執筆スタイル規約 | 単体: ステップ 4 / バッチ: Phase 2 Agent 起動時 |
| [references/agent-prompt-template.md](references/agent-prompt-template.md) | サブエージェント用プロンプト雛形 | バッチ: Phase 2 のみ |

---

## 注意事項

- **事実と主張を混ぜない**: 自分の解釈は wiki ではなく `my-research/drafts/` に書く
- 矛盾を見つけたら隠さず `## 議論` 節で明示する
- PDF 読み込みに失敗したらその論文をスキップし、ユーザーに報告する
- バッチモードでは **共有ファイルの更新は全て Phase 3 で一括実行**する（Agent 間の競合を避けるため）
- wiki ページの品質が最重要。テンプレートの各セクションを埋めるだけでなく、スタイル規約（日本語主役、初出用語の説明、具体例の挿入）を徹底する
