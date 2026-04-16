# 報告書フォーマット

lint-wiki の結果は以下の形式で報告する。

```markdown
# lint-wiki report [YYYY-MM-DD]

## 索引整合性
- ✅ / ❌ (詳細)

## frontmatter 不備
- wiki/papers/xxx.md — `tags` フィールドが欠落
- wiki/concepts/yyy.md — `description` フィールドが欠落

## テンプレート準拠
- wiki/papers/xxx.md — 5/8 セクション (要改善: 30秒具体例, 用語ガイド, 主要貢献 が欠落)
- 全体: N/M ページが準拠 (7/8 以上)

## tags ↔ concepts 不整合
- wiki/papers/xxx.md の tag `foo` に対応する concepts ページなし
- wiki/concepts/bar.md はどの paper の tags からも参照されていない

## リンク切れ
- wiki/papers/xxx.md → [[yyy]] (wiki/concepts/yyy.md が存在しない)

## 孤立ページ
- wiki/papers/xxx.md (index にも concepts にも未参照)

## 未解決の矛盾
- wiki/concepts/zzz.md §議論: ...

## カバレッジ不足
- my-research で言及されているが wiki/concepts/ に存在しない:
  - refuse-based-soundness
  - no-escape-predicate

## research positioning の鮮度
- current.md 最終更新: YYYY-MM-DD (vN)
- **パターン A (ズレあり・関連あり):**
  - wiki/papers/hydra.md (ingest: 2026-04-13) — §番号更新を推奨
- **パターン B (関連が薄くなった):**
  - wiki/papers/slacc.md (ingest: 2026-04-13) — 注釈追加を推奨

## bibkey 不整合
- wiki/papers/xxx.md の bibkey `aaa2020bbb` が refs.bib に存在しない

## 推奨アクション
1. ...
2. ...
```
