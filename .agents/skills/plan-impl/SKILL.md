---
name: plan-impl
description: >
  研究ストーリー (paper/research-story/main.tex) と shopy の現在の実装状態を照合し、
  次に実装すべきコンポーネントの具体的な計画を生成する。
  shopy の既存パターン (services/deprecated/, models/deprecated/) を参照して
  新規モジュール・Pydantic モデル・CLI コマンド・データフローの骨格を提案する。
  「実装計画を立てたい」「next step は何？」「shopy に何を追加すべき？」
  「研究ストーリーをコードに落とし込みたい」「〇〇を実装するには？」
  といったリクエストで発動。引数には実装したい概念や機能を自由記述で渡せる。
argument-hint: /plan-impl [実装したい概念や機能（自由記述、省略可）]
---

# plan-impl スキル

`paper/research-story/main.tex`（唯一の正式な研究記述）と shopy の現在の実装を突き合わせて、
次に実装すべきものの具体的な計画を作る。

研究詳細（アプローチの段階・シグナル・評価指標など）はスキル本体にハードコードせず、
**常に最新の `main.tex` から読み込む**。これにより研究ストーリーが変化しても計画の手順は変わらない。

## 実行手順

### Step 1: 研究ストーリーの読み込み

```
Read: paper/research-story/main.tex（全文）
```

以下のセクションを重点的に把握する：

- `\section{アプローチ（提案手法）}` — 実装対象の手法・段階
- `\section{Research Questions}` — RQ1/RQ2 と評価指標
- `\section{期待される貢献}` — `[ ]`（未完了）の項目
- `\todo{...}` マクロ — 未解決の実装・根拠不足箇所

### Step 2: shopy 現状の読み込み

```
ls: /Users/shoei-y/research/shopy/src/shopy/services/deprecated/
ls: /Users/shoei-y/research/shopy/src/shopy/models/deprecated/
Read: /Users/shoei-y/research/shopy/CLAUDE.md
```

既存のファイル名・クラス名・型注釈のパターンを把握する（新しい実装はこのパターンに従う）。

### Step 3: ギャップの特定

引数がある場合:
- 引数のキーワードを `main.tex` 内で Grep して関連節を特定
- その概念を実装するために必要なコンポーネントを列挙

引数がない場合:
- `main.tex` の `期待される貢献` セクションで `[ ]` の項目を列挙
- shopy の既存ファイル群と照合して未着手の実装を特定
- 依存関係（既存 Step の出力が次 Step の入力になるか）を考慮して優先順位を推薦

### Step 4: 実装計画の生成

以下のフォーマットで出力する（後述）。

---

## 出力フォーマット

```markdown
## 実装計画: <対象の概念・機能名>

### 対応する研究記述
- main.tex §<節名> (`main.tex:<行番号>`)
- 引用: "<原文の一部>"

### 新規ファイル

| ファイルパス | 役割 |
|---|---|
| `src/shopy/models/deprecated/<name>.py` | Pydantic モデル |
| `src/shopy/services/deprecated/<name>.py` | サービス層（パイプライン Step） |

### データモデル骨格

```python
# src/shopy/models/deprecated/<name>.py
class <NameRecord>(BaseModel):
    # 既存モデルから引き継ぐ共通フィールド
    repo: str
    method_class: str
    method_name: str
    # 新規フィールド
    <field>: <type>  # <説明>
```

### CLI コマンド

```bash
uv run shopy deprecated <subcommand> \
    --input data/... \
    --output data/...
```

### データフロー

```
<前 Step の出力ファイル>
  → <新 Step>
  → <新 Step の出力ファイル>
```

### テスト方針

- happy-path: <最低限確認するケース>
- edge-case: <注意が必要なケース>

### 着手の前提条件

- [ ] <先に完了していなければならない Step>
```

---

## 注意事項

- **研究詳細はスキルに書かない**。アプローチの段階・シグナル名・評価指標は `main.tex` から読む
- shopy の命名規則（`scan_deprecated.py`、`DeprecatedAddedRecord`、`lead_time_status` 等）を
  必ず `services/deprecated/` と `models/deprecated/` を読んでから踏襲する
- `main.tex` の「スコープ境界と反証可能性」節に反する実装（Javadoc を学習に使う等）は提案しない
- `\todo{...}` に書かれた未解決事項は「要確認」として計画に明示する
- データモデルは既存 Step の出力型をそのまま受け取れる形にする（型の互換性）
- 計画はあくまで骨格。実際の実装は `/check-research-alignment` で整合性を確認しながら進める

## 関連スキル

- `check-research-alignment`: 実装後の整合性チェック（本スキルは計画、あちらは検証）
- `revise-story`: 研究ストーリー自体を改訂したい場合
- `query-wiki`: 先行研究・概念の事実確認
