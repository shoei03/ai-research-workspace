# チェック項目の詳細手順

各チェックの具体的な実行方法。SKILL.md のチェックリストと対応する。

---

## 1. index.md の整合性

- `wiki/index.md` に列挙されているリンク先ファイルが実在するか（Glob で確認）
- `wiki/papers/` と `wiki/concepts/` の実ファイルが index にすべて載っているか

```bash
ls wiki/papers/*.md
ls wiki/concepts/*.md
```

index.md の内容と突合し、漏れ・不一致を検出する。

---

## 2. frontmatter 検証

各 `wiki/papers/*.md` の YAML frontmatter に必須フィールドが揃っているか:

| フィールド | 必須 | 例 |
|---|---|---|
| `title` | ✅ | `"LASE: Locating and ..."` |
| `authors` | ✅ | `[Na Meng, Miryung Kim, ...]` |
| `venue` | ✅ | `ICSE 2013` |
| `year` | ✅ | `2013` |
| `bibkey` | ✅ | `meng2013lase` |
| `tags` | ✅ | `[rewrite-rule-inference, ...]` |

各 `wiki/concepts/*.md` の frontmatter に `name`, `description`, `type` が揃っているか。

---

## 3. テンプレート準拠チェック

各 `wiki/papers/*.md` が wiki-paper-template.md の 8 主要セクションを備えているか Grep で確認:

1. `## 一行サマリー`
2. `## 30 秒で分かる具体例`
3. `## 問題意識`
4. `## 用語ガイド`
5. `## 主要貢献`
6. `## 手法`
7. `## 評価`
8. `## 限界`

7/8 以上を「準拠」、それ未満を「要改善」として報告。

---

## 4. tags ↔ concepts 整合性

papers の frontmatter `tags` に書かれた名前が `wiki/concepts/` に対応するファイルとして存在するか。

```
例: tags: [rewrite-rule-inference, parameterized-equivalence]
  → wiki/concepts/rewrite-rule-inference.md が存在するか
  → wiki/concepts/parameterized-equivalence.md が存在するか
```

逆方向も確認: concepts ページのうち、どの paper の tags からも参照されていないものを検出。

---

## 5. 関連 concept リンク検証

papers 末尾の `## 関連 concept` セクションにある `[[concept-name]]` のリンク先が実在するか。

---

## 6. 孤立ページ検出

- `wiki/papers/*.md` のうち、どの concept からもリンクされておらず、index.md にも載っていないページ
- `wiki/concepts/*.md` のうち、どの paper の tags にも `[[...]]` にも含まれていないページ

孤立ページは**即削除しない**。ユーザーに提示して判断を仰ぐ。

---

## 7. 矛盾検出

各 concept ページで `## 議論` や `## 矛盾` 節を探し、未解決の矛盾を一覧化。
ユーザーに「どちらの主張を採用するか、保留するか」を促す。

---

## 8. カバレッジ不足検出

`my-research/current.md` で言及される概念名を Grep し、それぞれに対応する `wiki/concepts/` ページが存在するか確認。

不足している概念は「my-research で主張しているが wiki に事実ページがない = 論文化時に引用する文献がない」という危険信号。

---

## 9. research positioning の鮮度チェック

wiki/papers の「本研究の立ち位置」系セクション（「本研究の立ち位置への含意」「本研究の立ち位置の再定義」等）が、**現在の** current.md と整合しているかを確認する。

current.md は研究の進行に伴い頻繁に更新され、古い版は `my-research/versions/` に移動する。wiki ページは ingest 時点の current.md に基づいて書かれるため、以下のズレが起きうる:

- current.md の §番号が変わった（§1 → §3 など）
- 主張そのものが変わった（ポジショニングの修正）
- 用語が変わった（概念名の改称）

**チェック手順:**

1. `my-research/current.md` の `date` フィールド（frontmatter）を確認
2. `log.md` の各 ingest 日付を確認
3. **current.md の date より前に ingest された paper** を「要確認」としてリストアップ
4. 該当 paper の「本研究の立ち位置」系セクションを抽出し、以下を確認:
   - 参照している §番号が current.md に存在するか
   - 言及しているキーワード（例: `refuse-based soundness`, `C1〜C4`, `N=1`）が current.md で同じ文脈で使われているか
   - current.md の versions/ にしか残っていない表現を参照していないか

5. ズレを検出したら、以下の 2 パターンに分類して報告する:

**パターン A: まだ関連あるが記述がズレている**
→ §番号や表現の更新を推奨

**パターン B: 研究の方向が変わり、直接の関連が薄くなった**
→ 元の記述は消さず、以下の注釈を先頭に追加することを推奨:

```markdown
> ⚠️ この記述は current.md vN (YYYY-MM-DD) 時点のもの。
> 現在の current.md (vM, YYYY-MM-DD) では直接の関連が薄い。
```

元の記述を残す理由: 将来の研究方針変更で再び関連する可能性がある。「かつてどう位置づけていたか」の記録としても価値がある。

---

## 10. bibkey 整合性

`wiki/papers/*.md` の frontmatter の `bibkey` と `paper/common/refs.bib` の BibTeX キーが一致しているか。

```bash
grep '^@' paper/common/refs.bib    # bib 側のキー一覧
```

papers 側の bibkey と突合し、片方にしかないものを検出。
