# IIIF Region & Tag Extension（草案）

IIIF Presentation API 4.0 に対する拡張の草案です。
3D（および 2D）の注釈について、次の 4 点を定めます。

1. **同じ場所に付いた複数の注釈を、同じ場所として扱う**
2. **タグに「キー」（分類軸）を持たせる**
3. **注釈に画像を添える（その場所の写真か、関連する外部の資料かを書き分ける）**
4. **注釈に関連資料を結び付ける（Wikidata は「何を表すか」、Wikipedia と文献は「関連する資料」）**

あわせて、注釈エディタが今出している形（`seeAlso` に入れる形）の読み方も書きました（仕様本文の §7）。

- 仕様本文（英語）: [spec/index.md](spec/index.md)
- 例: [examples/manifest.json](examples/manifest.json)（石淵家地球儀の 3D モデルに付けた例の注釈。文面は説明用）
- 著者: 中村 覚、小川 潤
- 状態: 草案（2026-09-28 作成）。小川さんと内容を調整中で、今後変わることがあります

## 背景

小川さんの注釈エディタ（[3D-annotation-viewer](https://github.com/junjun7613/3D-annotation-viewer)）では、
注釈を付ける前に「場所」（領域）を作り、そこに複数の注釈を結び付けます。
同じ場所に、別の人が別の解釈で注釈を付けられるようにするためです。
また、注釈にキーと値の組でタグを付けられます（例: 「身分: ローマ市民」）。

既存のビューア（Mirador、Universal Viewer など）はこの 2 点を知らないので、
同じ場所の注釈がばらばらに並び、タグのキーは表示されません。
この拡張は、それをどう読むかを定めます。

## どこが拡張なのか

基準は **IIIF Presentation API 4.0（4.0.0-draft）** です。版 4 が土台にしている
W3C Web Annotation Data Model（以下 W3C）の決まりも、標準として扱います。
2026-09-28 に各仕様の原文で確かめました。

| # | 項目 | 版 4 / W3C の決まり | 判定 |
|---|---|---|---|
| 1 | 注釈の対象（Specific Resource）に ID を付ける | 版 4 で**必須**。「Specific Resource must have an HTTP(S) URI given in id」（版 4 Data Model, Specific Resource）。版 3 と W3C では任意 | 標準 |
| 2 | 複数の注釈が**同じ ID** の Specific Resource を対象にしたら、同じ場所として扱う。ビューアはまとめて表示する | 定めなし | **拡張** |
| 3 | 同じ ID を共有する注釈は、どれも `source` と `selector` を省略せず、同じ値で書く | 定めなし | **拡張**（出す側の決まり） |
| 4 | 本文を配列にし、タグを `purpose: "tagging"` の TextualBody で書く | W3C で定義（TextualBody の `purpose`、値 `tagging`）。版 4 は W3C のクラスをそのまま使う | 標準 |
| 5 | 注釈の `motivation` に `tagging` を入れる | 版 4 で「配列でなければならない」。値は W3C の定義から選べる | 標準 |
| 6 | TextualBody に `label`（言語マップ）を付ける | 版 4 で TextualBody は `label` を持てる（may）。値は言語マップ | 標準 |
| 7 | タグの本文の `label` を、そのタグの**キー**として読む | 版 4 の `label` は「名前・見出し」の意味。キーとして読む決まりは無い | **拡張** |
| 8 | 3D の多角形を `WktSelector` と `POLYGON Z` で書く | 版 4 で定義 | 標準（拡張ではない。下の「直すところ」を参照） |
| 9 | 注釈の作成者・作成日時 | W3C の `creator` / `created` がある | 標準（拡張では定めない） |
| 11 | 本文の配列に `type: "Image"` の本文を入れ、画像サービス（`service`）と所蔵館の頁（`homepage`）を添える | W3C で定義（本文は複数持てる、本文に画像を使える）。`service`・`homepage` は版 4 で定義 | 標準 |
| 12 | 本文の `purpose` に `describing`（対象を記述する）・`linking`（関連する資料）・`identifying`（何を表すかを示す）を入れる | W3C で定義 | 標準 |
| 13 | `Image` の本文を画像として注釈と一緒に表示し、説明文やタグとしては扱わない。`describing` は場所の写真、`linking` は関連資料として分けて見せる | 定めなし | **拡張**（読み方だけ） |
| 14 | 本文の配列に、外部の資料（Wikidata の項目、Wikipedia の記事、文献）を `purpose` 付きで入れる | W3C で定義（本文に外部の資料を使える。`purpose` の値も W3C） | 標準 |
| 15 | Wikidata の項目は `identifying`、Wikipedia の記事と文献は `linking` で書く。ビューアは説明文・タグと分けて「関連資料」として見せる | 定めなし | **拡張**（読み方だけ） |
| 16 | URL の無い文献は、引用の文字列を `purpose: "linking"` の TextualBody で書く | 定めなし（W3C では TextualBody に `purpose` を付けられる） | **拡張**（読み方だけ） |
| 10 | マニフェストの `@context` に、この拡張の定義ファイルの URL を入れて使用を宣言する | 版 4 で「拡張の context は配列の先頭に、版 4 の context は最後に」と定める。定義ファイルの中身（用語）は拡張側で決める | **拡張**（用語は定義せず、目印として使う） |

つまり、**新しい用語は 1 つも増やしません。** 拡張が定めるのは
「同じ ID は同じ場所」（2、3）と「タグの `label` はキー」（7）と「`Image` の本文は画像として見せ、`purpose` で場所の写真か関連資料かを分ける」（13）と
「`identifying`・`linking` の本文は関連資料として見せる」（15、16）という読み方と、
その使用を宣言する目印（10）だけです。
そのため、この拡張を知らないビューアでも、注釈はこれまでどおり 1 件ずつ表示されます。

### 版 3 で出す場合

版 3 では Specific Resource の ID は任意です。そのため版 3 では、1 も
この拡張の決まりとして扱います（仕様本文の「Compatibility with Presentation 3.0」）。
打ち合わせで「ID を付けているところが既存の仕様と違う」と確かめたのは、この版 3 での話です。

## 直すところ（拡張ではなく、版 4 に合わせる修正）

仕様を書く過程で、エディタとビューアの食い違いが見つかりました。

| 項目 | エディタの出力 | ビューアが読む形 | 版 4 |
|---|---|---|---|
| 3D の多角形 | `PolygonZSelector` / `POLYGONZ((…))` | `WKTSelector` | `WktSelector` / `POLYGON Z (…)` |
| 説明文の `label` | 文字列 | 文字列 | 言語マップ（例: `{"ja": ["少年"]}`） |
| 拡張の宣言（`@context`） | 無し | 無くても読む | 配列の先頭に拡張の URL を入れる（拡張の決まり） |
| 注釈に添える画像 | `seeAlso` に `type: "Image"` と独自の `source`（画像の URL）を書く | `source` の画像を「その場所の写真」として出す（2026-11-01、iiif-3d-viewer #298） | `Image` の本文として書き、関連する外部の画像なら `purpose: "linking"` を付ける（`seeAlso` は機械が読む記述のための欄で、`source` はそこに定義が無い。W3C では `source` は「切り出す前の元の資料」を指す欄） |
| Wikidata・Wikipedia・文献 | `seeAlso` に入れる | `seeAlso` も本文も読む | 本文に入れ、Wikidata は `identifying`、Wikipedia と文献は `linking` を付ける |
| Wikidata の URI | 項目の頁の URL（`https://www.wikidata.org/wiki/Q144`） | どちらも読む | 項目そのものの URI（`http://www.wikidata.org/entity/Q144`）。頁の URL は「項目についての頁」を指す |
| 画像・文献の `id` | `<元のマニフェストの置き場所>/media/<uuid>`、`…/bibliography/<uuid>`。開くと 404 | 開かない。画像は `source` から取る | 資料そのものの URL を `id` に入れる |
| 文献の URL・DOI | マニフェストには無い。`/rdf` にだけある | まだ読まない | 本文の `id` に入れる（DOI は `https://doi.org/…`） |
| 領域の URI | マニフェストは `<base>/region/<uuid>`、`/rdf` は `<base>/annotation/region/<uuid>` | マニフェストの方だけ使う | 両方で同じ URI にする |
| `label` の言語 | 中身が日本語でも `en` | 特別な扱いはしない | 中身の言語を付ける（無ければ `none`） |
| タグが無いときの `motivation` | 文字列 `"commenting"` | 両方 | 配列 `["commenting"]` |

エディタは現在、版 3 のマニフェストを出力しています。版 3 では 3 行目の文字列は問題ありません。

## エディタの今の出力の読み方（仕様本文 §7 の要点）

エディタの出力を、拡張を知らないまま読むときのための説明です。2026-11-01 に、エディタのソース（`converter.ts`・`rdf.ts`）と、実際に配信されているマニフェストで確かめました。

- 関連資料は、注釈の `seeAlso` に入っています。画像・IIIF マニフェスト・動画・Sketchfab の模型は、**資料の URL が `source` に入っていて、`id` は開けません。**
- `id` の `/media/`・`/bibliography/`・`/region/` は、**元のマニフェストの置き場所の下に作られた名前**です（例: 元が `https://example.org/iiif/3/87/manifest` なら `https://example.org/iiif/3/87/media/<uuid>`）。エディタも元のサーバも配信していないので、開くと 404 になります。
- ただし、これらは `/rdf`（マニフェストの最上位の `seeAlso` のうち `format` が `text/turtle` のもの）の中の名前と同じです。**`/rdf` を引くための鍵として使えます。**
- `/rdf` にだけあるもの: 文献の URL・DOI・PDF、Wikidata の項目の画像、資料と注釈の関係の種類（「描いている」「背景を説明する」など）。逆に、画像の URL は `/rdf` には無く、マニフェストの `source` にだけあります。
- 関係の種類から `purpose` への対応は、仕様本文 §7.4 に**案**として表にしました。小川さんと相談して決めたい点です。
- `/rdf` が無いと、画像が「その場所の写真」か「関連する資料」かは分かりません。そのときは関連資料として見せるのが安全です（場所の写真と取り違えないため）。
