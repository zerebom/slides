---
theme: seriph
background: https://cover.sli.dev
class: 'text-center'
highlighter: shiki
lineNumbers: false
info: |
  ## Polarsの成長:0.14から1.0までの変遷と今後の展望
drawings:
  persist: false
transition: slide-left
title: Polarsの成長:v0.14からv1.0までの変遷と今後の展望
mdc: true

canvasWidth: 900

---

## Polarsの成長: v0.14からv1.0までの変遷と今後の展望
2024/07/12 Polars Data Crunch #2

Higuchi Kokoro

---

# 今日の内容

### ゴール
- Polarsのアップデートの歴史と将来像がわかる 💡
- Polarsに追加された便利な機能がキャッチアップできる ✨


<br>


### お品書き

1. これまでのPolarsのアップデートの歴史と傾向把握
2. アップデートのキャッチアップ
   1. Polars1.0と今後の展望
   1. 過去の重要なアップデートのサマリ

<br>

---
layout: image-left
image: /higu.png
---

# 自己紹介

- Higuchi Kokoro
  - Data Scientist@Commune
  - X: [@zerebom_3](https://x.com/zerebom_3)
- 担当領域: 推薦・検索・プロダクトのデータ分析・LLMなど
- ELDEN RINGにハマってます





---
layout: center
---
# Polarsのアップデートの歴史と変更の傾向


---

# 前準備: リリースのデータセット化

- リリースの傾向を定量評価するために、PyGitHub経由でRelease Noteを抽出
- 正規表現でカテゴリごとのPR数などを取得
- 今回はPythonのリリースのみを集計
- Ref: https://github.com/pola-rs/polars/releases


<img
  src="/release_note_big.png"
  alt=""
  class="m-5 h-70"
/>


---
layout: two-cols
---
# 取得コード

```python
from github import Github

g = Github(token)
repo = g.get_repo("pola-rs/polars")
releases = repo.get_releases()
data = []

for release in tqdm(releases):
    version = release.tag_name
    date = release.published_at
    data.append(
        {
            "Version": version,
            "Date": date.strftime("%Y-%m-%d"),
            "Month": date.strftime("%Y-%m"),
            "Description": release.body,
        }
    )

df = pl.DataFrame(data)

```

::right::
<br>
<br>

```python
release_notes = df["Description"].to_list()
pattern = r"##\s*([^\n]+)\s*((?:(?!##).|\n)*)"

dict_list = []
for note in release_notes:
    dic = {}
    matches = re.findall(pattern, note)
    for match in matches:
        dic[remove_emoji(match[0]).strip()] \
          = len(re.findall(r"- ", match[1]))
    dict_list.append(dic)

release_cnt_df = pl.from_dicts(dict_list)

df = (
    pl.concat(
      [df,release_cnt_df], how="horizontal"
      )
    .filter(pl.col("Language") == "py")
    .drop("Description")
)
```




---
layout: two-cols
---

# アップデートの頻度

- 2022/10: 0.14からリリース開始
- 各マイナーバージョンアップデート: 15-30回ほどのリリース
- リリース間隔: 初期は60日程度。直近は100-180日


::right::
<img
  src="/release_history.png"
  alt=""
  class="m-10"
/>


---

# アップデートの傾向

- PRのマージPR数が増加傾向
- Bug fixesが減少
- DeprecationsやEnhancementsの割合が増加

→ 徐々にライブラリが成熟していることを示唆していそう


<div style="display: flex; justify-content: space-around;">

<img
  src="/absolute.png"
  alt=""
  class="h-59"
/>

<img
  src="/percentage.png"
  alt=""
  class="h-59"
/>


</div>


---
layout: center
---
# Polars 1.0.0の変更点と今後の展望



---

# Polars 1.0.0の変更点と今後の展望
- GPU 対応 ⚡
  - NVIDIA RAPIDSによる高速化
  - `.collect(gpu=True)`で呼び出し可能にする予定、準備中
- Polars Cloudの導入 ☁️
  - ホスティングやスケーリングをしてくれるマネージドサービス
- 型の厳密化 📏
  - `pl.Series()`の初期化時に型が混合してるとError
  - `replace()は前後の型がマッチしていないとError

→ 総論、**大規模なデータをプロダクション品質で分析・処理できるライブラリ**に

refs: https://pola.rs/posts/announcing-polars-1/

---

# Polars 1.0.0への追従方法

- [Update Guide](https://pola-rs.github.io/polars/user-guide/misc/upgrade_guide/)を参照し、書き換える
- もしくは`polars-upgrade`ツールを使用し、自動変換する
  - pre-commit hookにも対応
  - ref: https://github.com/MarcoGorelli/polars-upgrade

<br>
<br>


```bash
pip install polars-upgrade
polars-upgrade my_project --target-version=1.0
```

---

# Versioning philosophy

- 1.0にアップデートしたものの、設計に欠陥があれば破壊的な変更を加える
   - ただし、頻度と深刻さは時間とともに大幅に減少すると予想
- RustのNightlyシステムに依存している箇所は、将来的に変更可能性あり
- 詳しいルールは[Versioning](https://docs.pola.rs/development/versioning/)を参照

---
layout: center
---

# 過去の重要なアップデート
https://pola.rs/posts/ の"Polars in Aggregate"をチェック!


---

# 外部ライブラリとの連携強化


<div class="grid grid-cols-2 gap-4">
<div>

## hvPlot連携

```python
df = pl.DataFrame({"x": range(10), "y": range(10)})
df.plot.line(x="x", y="y")
```

- 高レベル可視化ライブラリ`hvPlot`を直接呼び出せるように


<img
  src="/lineplot.webp"
  alt=""
  class="h-50"
/>



</div>
<div>

## PyTorch & JAX連携

```python
df = pl.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
torch_tensor = df.to_torch()
jax_array = df.to_jax()
```

- Tensorへ直接･高速に変換出来るように

## その他連携ライブラリ
- Great Tables(dfの可視化)
- Hugging Face
- scikit-learn


</div>
</div>

---

# APIの充実

<div class="grid grid-cols-2 gap-4">
<div>

### Struct操作の改善

```python
df.select(
    pl.col("item").struct.with_fields(
        pl.field("name").str.to_uppercase(),
        pl.field("car").fill_null("Mazda")
    )
)
```

- 構造体の属性に対して複数操作可能に

<br>

### Array操作の改善

```python
 df.with_columns(
  number_of_twos=pl.col("a").arr.count_matches(2)
)
```

- `count_matches`や`contains`などの便利なAPIが追加


</div>
<div>

### SQL機能の拡張

```python
df.sql("SELECT * FROM df WHERE a > 5")
```

```python
pl.sql("""
    SELECT df1.*, d
    FROM df1
    INNER JOIN df2 USING (a)
    WHERE a > 1 AND EXTRACT(year FROM c) < 2050
""").collect()
```

- `pl.sql()`では名前空間に定義された複数のdfを呼び出せる
- `df.sql()`でdfを直接操作可能
- `REGEXP`や`LIKE`等の構文も追加


</div>
</div>

---

# パフォーマンスの向上

- CSV/Excel書き込みの高速化
- `with_columns`の最適化

  ```python
  df.lazy()
        .with_columns(height_per_year=pl.col("height") / pl.col("age"))
        .with_columns(height_mean=pl.lit(0))
        .with_columns(age=pl.col("age") - 18)
        .with_columns(height_mean=pl.col("height").mean())
  ```

  - 複数のwith_columns()は内部で1つのクエリとして最適化

---

# まとめ
- Polars 1.0がリリースされた🎉
- 下記のような継続的改善が行われている
  - 機能拡張と性能改善
  - 外部ライブラリとの連携強化
  - ユーザーフレンドリーなAPIの進化

<br>

### 今後の展望

- GPU対応による更なる高速化
- Polars Cloudでのスケーラビリティ向上
- より型安全で堅牢なライブラリに




---

# 参考資料

- [Polars Blog](https://pola-rs.github.io/polars-book/)
- [GitHub Repository](https://github.com/pola-rs/polars)
- https://pola-rs.github.io/polars/user-guide/misc/upgrade_guide/

