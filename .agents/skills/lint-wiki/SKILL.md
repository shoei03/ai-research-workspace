---
name: lint-wiki
description: >
  wiki の健康診断。孤立ページ・リンク切れ・矛盾・カバレッジ不足・frontmatter 不備・
  テンプレート準拠度・research positioning の鮮度を検出する。
  wiki のチェック、整合性確認、品質確認、
  「wiki 大丈夫？」「リンク切れない？」「ページ足りてる？」のような依頼で使う。
disable-model-invocation: true
argument-hint: /lint-wiki
---

# lint-wiki スキル

`wiki/` の構造的健全性をチェックするスキル。ingest を繰り返すうちに発生するほつれを発見する。

## 前提知識

- papers の frontmatter: `title`, `authors`, `venue`, `year`, `bibkey`, `tags`
- papers の末尾: `## 関連 concept` に `[[concept-name]]` 形式でリンク
- concepts の frontmatter: `name`, `description`, `type`
- index.md: 通常の Markdown リンク `[text](path)` 形式
- テンプレート定義: [wiki-paper-template.md](../ingest-paper/references/wiki-paper-template.md)

---

## チェックリスト

以下の 10 項目を順にチェックする。各項目の詳細手順は [references/check-details.md](references/check-details.md) を参照。

| # | チェック | 概要 |
|---|---|---|
| 1 | index.md 整合性 | リンク先の実在 + 実ファイルの漏れ |
| 2 | frontmatter 検証 | papers/concepts の必須フィールド |
| 3 | テンプレート準拠 | wiki-paper-template.md の 8 セクション (7/8 以上で合格) |
| 4 | tags ↔ concepts | tags に書かれた concept が実在するか (双方向) |
| 5 | 関連 concept リンク | `[[concept-name]]` のリンク先が実在するか |
| 6 | 孤立ページ | どこからも参照されていないページ |
| 7 | 矛盾検出 | `## 議論` 節の未解決矛盾 |
| 8 | カバレッジ不足 | current.md の概念が wiki/concepts/ にあるか |
| 9 | positioning 鮮度 | 本研究の立ち位置記述が現在の current.md と整合するか |
| 10 | bibkey 整合性 | frontmatter の bibkey と refs.bib の突合 |

§9 が特に重要: current.md は頻繁に更新されるため、wiki の positioning 記述が陳腐化しやすい。ズレの検出時は **パターン A** (関連あり・記述ズレ → 更新推奨) と **パターン B** (関連薄 → 注釈追加推奨) に分類する。詳細は [references/check-details.md](references/check-details.md) §9 を参照。

---

## 報告書

[references/report-template.md](references/report-template.md) のフォーマットに従って結果を報告する。

---

## references

| ファイル | 内容 |
|---|---|
| [references/check-details.md](references/check-details.md) | 各チェック項目の詳細手順 |
| [references/report-template.md](references/report-template.md) | 報告書のフォーマット |

---

## 注意事項

- **自動修正はしない**。検出と提案に留める
- 孤立ページの削除はユーザー許可が必要
- 定期的に（月1程度、または大量 ingest 後に）走らせる運用を想定
