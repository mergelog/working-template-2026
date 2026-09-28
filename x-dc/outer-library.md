# 00_12. 外部プラグイン（用途と使い方の例）

## この資料の使い方

外部ライブラリでできることを、用途別の一覧と小さなコード例でまとめる。データの比較・日付の整形・検索・ファイル出力など、作りたい処理から使い方を探せる。

- 関数・クラス・型ごとに、処理目的・使い方・注意点を掲載する
- 各表の「詳細」から、対応するコード例へ移動できる
- コード例にはユーザー情報・設定・一覧・グラフなどの汎用的なデータを使う
- `isValidQuery` は、`lucene.parse` を組み合わせる自作関数の例として掲載する
- `@angular/*` / `@ngrx/*` / `rxjs` / `primeng` / `chart.js` などの基盤ライブラリは別途扱う

---

## 1. lodash-es（汎用ユーティリティ / 34 関数）

ESM 版 lodash。型は `@types/lodash-es`。
**「値の比較」「安全な取り出し」「深いコピー」** など、配列・オブジェクト・文字列の処理を短く書ける。
例えば、設定の変更を判定したり、元データを保ったまま編集用のコピーを作ったりするときに使う。

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| lodash-es | `isEqual` | 比較 | 2 値を再帰的に深く比較。設定の変更判定や `distinctUntilChanged` の比較に使える | [詳細](#api-001) |
| lodash-es | `get` | オブジェクト | `a.b[0].c` のようなパスを安全に取得。存在しなければ既定値 | [詳細](#api-002) |
| lodash-es | `cloneDeep` | オブジェクト | 深いコピーを作る。元データを変更せず編集・加工するため | [詳細](#api-003) |
| lodash-es | `merge` | オブジェクト | ネストしたオブジェクトを再帰マージ（破壊的） | [詳細](#api-004) |
| lodash-es | `last` | 配列 | 配列の最後の要素を取得 | [詳細](#api-005) |
| lodash-es | `pick` | オブジェクト | 指定キーだけを抜き出した新オブジェクトを作る | [詳細](#api-006) |
| lodash-es | `uniqBy` | 配列 | 判定関数の結果が同じ要素を除いて一意化（id 単位の重複排除） | [詳細](#api-007) |
| lodash-es | `uniq` | 配列 | 値の重複を除いて一意化 | [詳細](#api-008) |
| lodash-es | `isUndefined` | 型判定 | `undefined` か判定（`null` は false） | [詳細](#api-009) |
| lodash-es | `flatten` | 配列 | 1 段だけ平坦化 | [詳細](#api-010) |
| lodash-es | `camelCase` | 文字列 | `foo_bar` → `fooBar`。API の snake_case を JS 側へ変換 | [詳細](#api-011) |
| lodash-es | `isEmpty` | 型判定 | 空判定。`{}` `[]` `''` `null` をまとめて見る | [詳細](#api-012) |
| lodash-es | `isNil` | 型判定 | `null` または `undefined` か判定 | [詳細](#api-013) |
| lodash-es | `escapeRegExp` | 文字列 | 正規表現の特殊文字をエスケープ。検索語を安全に RegExp 化 | [詳細](#api-014) |
| lodash-es | `sortBy` | コレクション | キー抽出関数で安定ソート（常に昇順） | [詳細](#api-015) |
| lodash-es | `capitalize` | 文字列 | 先頭だけ大文字、残りは小文字。ラベル表示用 | [詳細](#api-016) |
| lodash-es | `unionBy` | 配列 | 複数配列を判定関数基準で重複なく結合 | [詳細](#api-017) |
| lodash-es | `escape` | 文字列 | `&<>"'` を HTML エンティティへ変換。HTML サニタイズの代用ではない | [詳細](#api-018) |
| lodash-es | `min` | 数値 | 配列の最小値 | [詳細](#api-019) |
| lodash-es | `set` | オブジェクト | パス指定で値を書き込む（途中の階層は自動生成・破壊的） | [詳細](#api-020) |
| lodash-es | `has` | オブジェクト | パスがキーとして存在するか判定 | [詳細](#api-021) |
| lodash-es | `castArray` | 配列 | 単値でも配列でも必ず配列にそろえる | [詳細](#api-022) |
| lodash-es | `mergeWith` | オブジェクト | マージ規則を自前関数で指定できる `merge`（配列の扱いを変えたい時） | [詳細](#api-023) |
| lodash-es | `isArray` | 型判定 | 配列か判定（`Array.isArray` 相当） | [詳細](#api-024) |
| lodash-es | `parseInt` | 数値 | 基数を厳密に扱う `parseInt`。環境差を吸収 | [詳細](#api-025) |
| lodash-es | `template` | 関数 | 文字列テンプレートをコンパイルして関数化 | [詳細](#api-026) |
| lodash-es | `omit` | オブジェクト | 指定キーを除いた新オブジェクトを作る（`pick` の逆） | [詳細](#api-027) |
| lodash-es | `findLastIndex` | 配列 | 条件に合う最後の要素の index を返す | [詳細](#api-028) |
| lodash-es | `max` | 数値 | 配列の最大値 | [詳細](#api-029) |
| lodash-es | `zip` | 配列 | 複数配列を index ごとに組にまとめる | [詳細](#api-030) |
| lodash-es | `chunk` | 配列 | 指定サイズごとに分割（バッチリクエスト用） | [詳細](#api-031) |
| lodash-es | `slice` | 配列 | 範囲切り出し。疎配列も正しく扱う | [詳細](#api-032) |
| lodash-es | `snakeCase` | 文字列 | `fooBar` → `foo_bar`。API へ送る時の変換 | [詳細](#api-033) |
| lodash-es | `orderBy` | コレクション | 複数キー＋昇降順を指定してソート（`sortBy` の上位版） | [詳細](#api-034) |

---

## 2. ngxtension（Angular Signal 用ユーティリティ）

lodash が「JS の汎用」なら、こちらは「Angular Signal の汎用」。
**サブパス単位で import** するのが作法（`ngxtension/xxx`）。

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| ngxtension | `computedPrevious` | Signal | Signal の「前回値」を保持した Signal を作る。差分検知やアニメーション起点に | [詳細](#api-035) |
| ngxtension | `injectResize` | DOM 購読 | 要素サイズ変更を `Observable` で購読（ResizeObserver のラッパー） | [詳細](#api-036) |
| ngxtension | `explicitEffect` | Signal | 依存 Signal を明示指定する `effect`。暗黙依存による意図しない再実行を防ぐ | [詳細](#api-037) |
| ngxtension | `injectQueryParams` | ルーティング | URL のクエリパラメータを Signal 化。`?x=1` の変化に追従 | [詳細](#api-038) |
| ngxtension | `injectParams` | ルーティング | ルートパラメータ（`:id` など）を Signal 化 | [詳細](#api-039) |
| ngxtension | `NgxResize` / `ResizeResult` | DOM 購読 | ディレクティブ版のリサイズ購読と、その結果型 | [詳細](#api-040) |
| ngxtension | `injectRouteData` | ルーティング | ルートの `data` を Signal 化 | [詳細](#api-041) |
| ngxtension | `mapArray` | RxJS | 配列ストリームの各要素に `map` を適用する演算子（`map(arr => arr.map(f))` の短縮） | [詳細](#api-042) |

---

## 3. 日付・ID・色などの用途別ユーティリティ

### 3-1. date-fns（日付）

関数単位 import の日付ライブラリ。immutable で、moment と違い元の `Date` を壊さない。

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| date-fns | `format` | 整形 | `Date` を指定パターンの文字列へ（`yyyy-MM-dd HH:mm` など） | [詳細](#api-043) |
| date-fns | `parseISO` | 解析 | ISO 8601 文字列を `Date` に変換。API のタイムスタンプ受け取り | [詳細](#api-044) |
| date-fns | `addDays` | 加算 | 日数を加算した新しい `Date` を返す | [詳細](#api-045) |
| date-fns | `startOfDay` | 丸め | その日の 00:00:00 に丸める。期間フィルタの下限に使う | [詳細](#api-046) |
| date-fns | `subSeconds` | 減算 | 秒を引く。「直近 N 秒」の範囲計算 | [詳細](#api-047) |
| date-fns | `isValid` | 判定 | 有効な日付か判定（`Invalid Date` 検出） | [詳細](#api-048) |
| date-fns | `formatDistance` | 整形 | 「3 分」のような距離を表現。`addSuffix` で「前」も付けられる | [詳細](#api-049) |
| date-fns | `enGB`（locale） | ロケール | 英国式ロケール。日付の曜日・順序の基準 | [詳細](#api-050) |

### 3-2. uuid（ID 生成）

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| uuid | `v4` | ランダム | 乱数ベースの UUID。一時 ID・DOM の一意キー生成 | [詳細](#api-051) |
| uuid | `v5` | ハッシュ | 名前空間＋名前から **決定的に** 同じ UUID を作る。同入力→同 ID が必要な場面 | [詳細](#api-052) |
| uuid | `v1` | 時刻 | タイムスタンプベース。文字列の単純ソートは時系列順を保証しない | [詳細](#api-053) |

### 3-3. @ctrl/tinycolor（色）

`tinycolor2` の TypeScript 移植。`new TinyColor(色)` を起点にメソッドチェーンする。

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| @ctrl/tinycolor | `new TinyColor()` | 生成 | hex / rgb オブジェクト / hsl 文字列など雑多な入力を 1 つの色オブジェクトへ正規化 | [詳細](#api-054) |
| @ctrl/tinycolor | `.toRgb()` | 変換 | `{r,g,b,a}` オブジェクトで取り出す | [詳細](#api-055) |
| @ctrl/tinycolor | `.toHexString()` | 変換 | `#rrggbb` 文字列へ。設定保存・CSS 出力用 | [詳細](#api-056) |
| @ctrl/tinycolor | `.toRgbString()` | 変換 | `rgb(r, g, b)` 文字列へ | [詳細](#api-057) |
| @ctrl/tinycolor | `.mix()` | 加工 | 2 色を指定比率で混ぜる。系列色のグラデーション生成 | [詳細](#api-058) |
| @ctrl/tinycolor | `.lighten()` | 加工 | 明度を上げる（グラフの枠線色など） | [詳細](#api-059) |
| @ctrl/tinycolor | `.shade()` | 加工 | 黒を混ぜて暗くする | [詳細](#api-060) |
| @ctrl/tinycolor | `.setAlpha()` | 加工 | 透明度を設定 | [詳細](#api-061) |
| @ctrl/tinycolor | `.toHsl()` | 変換 | `{h,s,l}` で取り出す。色相だけずらす処理に使う | [詳細](#api-062) |
| @ctrl/tinycolor | `.isDark()` | 判定 | 暗い色か判定。文字色を白/黒どちらにするかの分岐 | [詳細](#api-063) |
| @ctrl/tinycolor | `.getLuminance()` | 計測 | WCAG 相対輝度を返す。コントラスト計算の土台 | [詳細](#api-064) |
| @ctrl/tinycolor | `.getBrightness()` | 計測 | 0〜255 の知覚的明るさ | [詳細](#api-065) |
| @ctrl/tinycolor | `mostReadable` | 判定 | 背景色に対し候補色の中から最も読みやすい色を選ぶ。タグ(chip)の文字色決定 | [詳細](#api-066) |

### 3-4. dompurify（HTML サニタイズ）

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| dompurify | `DOMPurify.sanitize` | セキュリティ | HTML から script などの危険要素を除去。`ADD_TAGS`/`FORBID_ATTR` で許可範囲を制御 | [詳細](#api-067) |
| dompurify | `DOMPurify.addHook` | セキュリティ | サニタイズの指定段階に処理を追加。例えば `iframe` の `src` を検査する | [詳細](#api-068) |

### 3-5. semver（バージョン比較）

`semver/functions/xxx` のサブパス import で、パッケージ全体は取り込まない。

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| semver | `valid` | 検証 | セマンティックバージョンとして妥当か（不正なら `null`）。フォーム検証に使用 | [詳細](#api-069) |
| semver | `gt` | 比較 | `a > b` 判定。サーバ版数が要求より新しいかの判定 | [詳細](#api-070) |
| semver | `gte` | 比較 | `a >= b` 判定 | [詳細](#api-071) |

### 3-6. filesize（バイト数整形）

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| filesize | `filesize` | 整形 | バイト数を単位付き文字列へ整形。ファイルサイズやグラフ軸ラベルの表示に使える | [詳細](#api-072) |
| filesize | `FilesizeOptions` | 型 | 単位系・小数桁などのオプション型 | [詳細](#api-073) |

### 3-7. @ngneat/dag（DAG レイアウト計算）

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| @ngneat/dag | `DagManagerService` | グラフ | ノード配列から DAG の段組み（何段目の何番目か）を計算するサービス。処理フロー図の土台 | [詳細](#api-074) |
| @ngneat/dag | `DagModelItem` | 型 | DAG ノードの必須型（`stepId` / `parentIds` / `branchPath`）。自前の型が extends する | [詳細](#api-075) |

---

## 4. 検索・表示・出力などの用途別ライブラリ

| プラグイン名 | 関数名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| marked | `marked.parse` | Markdown | Markdown を HTML に変換。説明文やプレビューの表示に使える | [詳細](#api-076) |
| diff | `Diff.diffArrays` | 差分 | 2 配列を厳密等価で比較し、追加/削除/共通のブロック列を返す。一覧や設定の変更箇所をハイライトする | [詳細](#api-077) |
| lucene | `lucene.parse` | 構文解析 | Lucene クエリ文字列を AST に変換。検索条件の解析 | [詳細](#api-078) |
| lucene（組み合わせ例） | `isValidQuery`（自作） | 構文解析 | `lucene.parse` の構文エラーを真偽値に変換し、検索欄の入力を検証する | [詳細](#api-079) |
| hocon-parser | `parseHocon` | 構文解析 | HOCON（Typesafe Config 形式）文字列を JS オブジェクトへ。アプリケーション設定の読み取り | [詳細](#api-080) |
| export-to-csv | `mkConfig` | CSV | ファイル名・ヘッダ有無などの CSV 設定オブジェクトを作る | [詳細](#api-081) |
| export-to-csv | `generateCsv` | CSV | 設定を部分適用した関数に行データを渡して CSV を生成（カリー化 API） | [詳細](#api-082) |
| export-to-csv | `asString` | CSV | 生成結果を文字列として取り出す（クリップボード用） | [詳細](#api-083) |
| export-to-csv | `download` | CSV | 生成結果をブラウザのダウンロードとして保存させる | [詳細](#api-084) |
| dom-to-image | `domtoimage.toBlob` | 画像化 | DOM 要素を SVG 経由でラスタ画像の `Blob` に変換。グラフのダウンロード | [詳細](#api-085) |
| ansi-to-html | `new Convert()` | ログ整形 | ANSI エスケープ変換器のインスタンス生成（色設定を保持） | [詳細](#api-086) |
| ansi-to-html | `convert.toHtml` | ログ整形 | ANSI カラーコードを `<span style>` 付き HTML へ。コンソールログ表示 | [詳細](#api-087) |
| has-ansi | `hasAnsi` | ログ整形 | 文字列に ANSI エスケープが含まれるか判定。変換が要るかの事前チェック | [詳細](#api-088) |
| string-to-color | `stc` | 色 | 任意の文字列から決定的に色を決める。ラベル名→色の自動割り当て | [詳細](#api-089) |
| d3-selection | `select` | DOM | セレクタ/要素から d3 セレクションを作る。`.on('click', ...)` でイベント購読。グラフ内の要素などへ処理を追加する | [詳細](#api-090) |
| d3-interpolate | `interpolateBasis` | 補間 | 値の配列を通る B スプライン補間関数を作る。グラフの平滑化 | [詳細](#api-091) |
| curved-arrows | `getBoxToBoxArrow` | 座標計算 | 2 つの矩形の間を結ぶ曲線矢印の制御点を算出。処理フロー図のノード間矢印 | [詳細](#api-092) |
| url | `parse` | URL 解析 | Node 互換の `url.parse` で URL のホストやパスを取り出す | [詳細](#api-093) |
| url | `UrlWithStringQuery` | 型 | `parse` の戻り値型 | [詳細](#api-094) |

---

## 5. 型定義の利用例

型を使って、ライブラリへ渡すデータの形を確認する例。

| プラグイン名 | 型名 | カテゴリ | 処理目的 | 詳細 |
| --- | --- | --- | --- | --- |
| plotly.js | `PlotData` | 型 | グラフの系列データを型付きで組み立てる | [詳細](#api-095) |

---

## 6. 用途から読む

1. **データの比較・編集** — lodash-es の `isEqual` / `get` / `cloneDeep` / `merge` / `pick` / `uniq` / `uniqBy`
2. **画面の状態・サイズ・URL への追従** — ngxtension の `computedPrevious` / `explicitEffect` / `injectResize` / 各種ルーティングユーティリティ
3. **日付・期間の表示と計算** — date-fns の `format` / `parseISO` / `addDays` / `formatDistance`
4. **検索・ファイル出力・グラフなど** — 必要な用途の表から対応する例を読む

---

## 7. 各項目の詳細な使い方

各表の「詳細」から対応する項目へ移動できる。各例は、引数・戻り値・用途を小さな値で示す。

### 7-1. lodash-es

以下の例は、必要な関数を `import {isEqual} from 'lodash-es'` のように個別 import して使う。

<a id="api-001"></a>

#### `isEqual`

**使い方**

```ts
isEqual({ tags: ['a'] }, { tags: ['a'] }) // true
```

2 つの値を渡し、配列・オブジェクトの中身まで等しいとき `true`。参照が異なるオブジェクトも同じ内容なら等しい。例えば、選択値や設定の内容が変わったかを判定するときに使う。大きなデータに毎回適用すると比較コストが増える。

[公式資料](https://lodash.com/docs/4.17.15#isEqual)

<a id="api-002"></a>

#### `get`

**使い方**

```ts
const user = { profile: { name: 'A' } };
get(user, 'profile.name', '未設定') // 'A'
get(user, 'profile.city', '未設定') // '未設定'
```

第 1 引数は対象、第 2 引数はパス、第 3 引数は省略可能な既定値。パスは `['profile', 'name']` とも書ける。既定値が使われるのは結果が `undefined` のときで、`null` はそのまま返る。

[公式資料](https://lodash.com/docs/4.17.15#get)

<a id="api-003"></a>

#### `cloneDeep`

**使い方**

```ts
const copy = cloneDeep({ settings: { theme: 'light' } });
copy.settings.theme = 'dark';
```

ネストした配列・オブジェクトも複製するので、元の `settings.theme` は `light` のまま。取得済みデータから編集用のコピーを作るときに使える。クラスインスタンスや DOM ノードなどは期待どおり複製できるか別途確認する。

[公式資料](https://lodash.com/docs/4.17.15#cloneDeep)

<a id="api-004"></a>

#### `merge`

**使い方**

```ts
const combined = merge({}, { user: { name: 'A' } }, { user: { role: 'admin' } });
```

後ろのオブジェクトを前へ再帰的に統合し、例では `user` が `{ name: 'A', role: 'admin' }` になる。**第 1 引数を変更する**ため、状態管理では例のように `{}` を先頭に置く。配列は連結せず添字ごとにマージされる。

[公式資料](https://lodash.com/docs/4.17.15#merge)

<a id="api-005"></a>

#### `last`

**使い方**

```ts
last(['初回', '最新']) // '最新'
```

配列の末尾 1 件を返す。空配列なら `undefined`。例えば、履歴配列から最新の項目を取り出すときに使う。

[公式資料](https://lodash.com/docs/4.17.15#last)

<a id="api-006"></a>

#### `pick`

**使い方**

```ts
pick({ id: 1, name: 'A', secret: 'x' }, ['id', 'name']) // { id: 1, name: 'A' }
```

第 2 引数で残したいキーを指定し、新しいオブジェクトを作る。元データは変更しないが、ネストした値のコピーは浅い。

[公式資料](https://lodash.com/docs/4.17.15#pick)

<a id="api-007"></a>

#### `uniqBy`

**使い方**

```ts
uniqBy([{ id: 1, name: 'A' }, { id: 1, name: 'B' }], 'id') // A 側だけ
```

各要素から `id` を取り、同じ値の重複を除く。判定関数 `item => item.id` も渡せる。最初に現れた要素が残るため、後勝ちにしたいなら並べ替えが必要。

[公式資料](https://lodash.com/docs/4.17.15#uniqBy)

<a id="api-008"></a>

#### `uniq`

**使い方**

```ts
uniq(['a', 'b', 'a']) // ['a', 'b']
```

値が同じ要素を 1 つにする。出現順を保ち、最初の値が残る。オブジェクトは中身でなく参照で比較するため、ID 単位なら `uniqBy` を使う。

[公式資料](https://lodash.com/docs/4.17.15#uniq)

<a id="api-009"></a>

#### `isUndefined`

**使い方**

```ts
isUndefined(undefined) // true; isUndefined(null) // false
```

引数が厳密に `undefined` かだけを判定。`null` もまとめて空扱いしたい場合は `isNil`。

[公式資料](https://lodash.com/docs/4.17.15#isUndefined)

<a id="api-010"></a>

#### `flatten`

**使い方**

```ts
flatten([[1, 2], [3]]) // [1, 2, 3]
```

配列の入れ子を **1 段だけ** 展開する。`[[[1]]]` なら `[[1]]` が残る。グループごとに分かれた項目を 1 つの配列にまとめるときに使える。

[公式資料](https://lodash.com/docs/4.17.15#flatten)

<a id="api-011"></a>

#### `camelCase`

**使い方**

```ts
camelCase('created_at') // 'createdAt'
```

区切りや大文字小文字を整えて camelCase に変換。API の snake_case 名を画面側のプロパティ名へ寄せる場合に使う。

[公式資料](https://lodash.com/docs/4.17.15#camelCase)

<a id="api-012"></a>

#### `isEmpty`

**使い方**

```ts
isEmpty({}) // true; isEmpty([]) // true; isEmpty('') // true
```

オブジェクトなら列挙可能な自身のキー、配列・文字列なら長さを見て空判定する。`null` や数値も `true` になるので、`0` を「データなし」と誤判定したくない場面では型を先に限定する。

[公式資料](https://lodash.com/docs/4.17.15#isEmpty)

<a id="api-013"></a>

#### `isNil`

**使い方**

```ts
isNil(null) // true; isNil(undefined) // true; isNil(0) // false
```

`null` と `undefined` だけをまとめて判定。空文字・`false`・`0` は有効値として残す。

[公式資料](https://lodash.com/docs/4.17.15#isNil)

<a id="api-014"></a>

#### `escapeRegExp`

**使い方**

```ts
new RegExp(escapeRegExp('a.b')) // ドットを文字どおり検索
```

ユーザー入力を正規表現のパターンへ埋め込む前に、`.` や `*` などの記号をエスケープする。正規表現の構文として解釈させたい入力には使わない。

[公式資料](https://lodash.com/docs/4.17.15#escapeRegExp)

<a id="api-015"></a>

#### `sortBy`

**使い方**

```ts
sortBy([{ name: 'B' }, { name: 'A' }], 'name') // A, B の順
```

キー名や `item => item.name` を指定して昇順に並べ、新しい配列を返す。複数キーは `['group', 'name']`。降順を混ぜるなら `orderBy`。

[公式資料](https://lodash.com/docs/4.17.15#sortBy)

<a id="api-016"></a>

#### `capitalize`

**使い方**

```ts
capitalize('hELLO') // 'Hello'
```

先頭を大文字にし、残りを小文字にする。`HELLO` を `Hello` にするので、略語をそのまま維持したい表示には注意。

[公式資料](https://lodash.com/docs/4.17.15#capitalize)

<a id="api-017"></a>

#### `unionBy`

**使い方**

```ts
unionBy([{ id: 1 }], [{ id: 1 }, { id: 2 }], 'id') // id 1, 2
```

複数配列を左から順に見て、指定キーまたは関数の結果が重複する要素を除きながら結合する。ID が同じなら先に出た要素が残る。

[公式資料](https://lodash.com/docs/4.17.15#unionBy)

<a id="api-018"></a>

#### `escape`

**使い方**

```ts
escape('<b>Tom & Sue</b>') // '&lt;b&gt;Tom &amp; Sue&lt;/b&gt;'
```

テキスト中の `&`, `<`, `>`, 引用符を HTML エンティティにする。HTML としてではなく文字どおり表示したい場合に使う。HTML 全体のサニタイズや URL 検証の代わりにはならない。

[公式資料](https://lodash.com/docs/4.17.15#escape)

<a id="api-019"></a>

#### `min`

**使い方**

```ts
min([8, 3, 5]) // 3
```

数値配列から最小値を返す。空配列なら `undefined`。日時などのキーを比較したい場合は別の集約処理が必要。

[公式資料](https://lodash.com/docs/4.17.15#min)

<a id="api-020"></a>

#### `set`

**使い方**

```ts
const next = set({}, 'user.name', 'A'); // { user: { name: 'A' } }
```

パスの途中がなければ作成して値を書き込む。**第 1 引数を変更する**ため、既存 state を直接渡さずコピーを渡す。信頼できない文字列をパスに使う設計は避ける。

[公式資料](https://lodash.com/docs/4.17.15#set)

<a id="api-021"></a>

#### `has`

**使い方**

```ts
has({ user: { name: null } }, 'user.name') // true
```

指定パスに **自身のプロパティ** が存在するか判定する。値が `null` や `undefined` でも、キーがあれば `true`。値を取得したいときは `get`。

[公式資料](https://lodash.com/docs/4.17.15#has)

<a id="api-022"></a>

#### `castArray`

**使い方**

```ts
castArray('a') // ['a']; castArray(['a']) // ['a']
```

単値を 1 要素配列へ、配列をそのまま返す。既存配列はコピーされないので、後から変更するなら別途複製する。引数なしは `[]`。

[公式資料](https://lodash.com/docs/4.17.15#castArray)

<a id="api-023"></a>

#### `mergeWith`

**使い方**

```ts
mergeWith({}, { tags: ['a'] }, { tags: ['b'] }, (left, right) => Array.isArray(left) ? left.concat(right) : undefined)
```

最後の関数でマージ規則を上書きする。例では配列を `['a', 'b']` に連結し、それ以外は `undefined` を返して通常の `merge` に任せる。第 1 引数は変更される。

[公式資料](https://lodash.com/docs/4.17.15#mergeWith)

<a id="api-024"></a>

#### `isArray`

**使い方**

```ts
isArray([1, 2]) // true; isArray({ 0: 1 }) // false
```

配列かどうかを判定する。`Array.isArray` と同じ用途で、オブジェクトや文字列とは区別できる。

[公式資料](https://lodash.com/docs/4.17.15#isArray)

<a id="api-025"></a>

#### `parseInt`

**使い方**

```ts
parseInt('08', 10) // 8
```

文字列を指定した基数で整数へ変換。小数部分は切り捨てられ、不正な文字列は `NaN`。10 進数で扱うなら第 2 引数を明示すると読みやすい。

[公式資料](https://lodash.com/docs/4.17.15#parseInt)

<a id="api-026"></a>

#### `template`

**使い方**

```ts
const render = template('Hello <%- name %>!');
render({ name: '<A>' }) // 'Hello &lt;A&gt;!'
```

テンプレート文字列から関数を作る。`<%= name %>` は補間、`<%- name %>` は HTML エスケープ付き補間。テンプレート自体に信頼できないコードを渡さない。例えば、共通の書式からラベルやメッセージを組み立てるときに使う。

[公式資料](https://lodash.com/docs/4.17.15#template)

<a id="api-027"></a>

#### `omit`

**使い方**

```ts
omit({ id: 1, name: 'A', secret: 'x' }, ['secret']) // secret 以外
```

指定キーを除いた新しいオブジェクトを返す。残したいキーを列挙する `pick` と使い分ける。ネストした値の参照を共有する点に注意。

[公式資料](https://lodash.com/docs/4.17.15#omit)

<a id="api-028"></a>

#### `findLastIndex`

**使い方**

```ts
findLastIndex([1, 4, 3, 6], n => n % 2 === 0) // 3
```

右端から条件に合う要素を探し、見つかった添字を返す。見つからなければ `-1`。要素そのものが欲しいなら `findLast` 系を使う。

[公式資料](https://lodash.com/docs/4.17.15#findLastIndex)

<a id="api-029"></a>

#### `max`

**使い方**

```ts
max([8, 3, 5]) // 8
```

数値配列の最大値。空配列なら `undefined`。

[公式資料](https://lodash.com/docs/4.17.15#max)

<a id="api-030"></a>

#### `zip`

**使い方**

```ts
zip(['A', 'B'], [10, 20]) // [['A', 10], ['B', 20]]
```

複数配列の同じ添字を 1 組にする。長さが違う場合は長い方に合わせ、不足分は `undefined`。対応するラベルと値を束ねるときに便利。

[公式資料](https://lodash.com/docs/4.17.15#zip)

<a id="api-031"></a>

#### `chunk`

**使い方**

```ts
chunk([1, 2, 3, 4, 5], 2) // [[1, 2], [3, 4], [5]]
```

第 2 引数の件数ごとに配列を分割する。API の一括処理上限に合わせて分ける用途。サイズは 1 以上を指定する。

[公式資料](https://lodash.com/docs/4.17.15#chunk)

<a id="api-032"></a>

#### `slice`

**使い方**

```ts
slice(['a', 'b', 'c'], 1, 3) // ['b', 'c']
```

開始位置を含み、終了位置を含まない範囲を新しい配列で返す。元配列は変えない。`Array.prototype.slice` と同じ感覚で読める。

[公式資料](https://lodash.com/docs/4.17.15#slice)

<a id="api-033"></a>

#### `snakeCase`

**使い方**

```ts
snakeCase('createdAt') // 'created_at'
```

単語の境界を `_` 区切りの小文字へ変換する。camelCase のプロパティ名を API の snake_case 形式へ合わせるときに使う。

[公式資料](https://lodash.com/docs/4.17.15#snakeCase)

<a id="api-034"></a>

#### `orderBy`

**使い方**

```ts
orderBy([{ name: 'A', score: 2 }, { name: 'B', score: 3 }], ['score'], ['desc']) // B, A
```

第 2 引数に並べ替えキーの配列、第 3 引数に対応する `asc` / `desc` を渡す。複数キーなら例: `['group', 'score']`, `['asc', 'desc']`。新しい配列を返す。

[公式資料](https://lodash.com/docs/4.17.15#orderBy)

### 7-2. ngxtension

Signal / effect の例はコンポーネントのフィールドまたはコンストラクタ内で使う想定。`const` を使う例は注入コンテキスト内で実行する。各項目に import 先を記す。

<a id="api-035"></a>

#### `computedPrevious`

**使い方**

```ts
const count = signal(1);
const previous = computedPrevious(count);
previous() // 1（初回）
count.set(2);
previous() // 1（変更前の値）
```

現在値を返す関数または Signal を渡し、前回値を表す `Signal<T>` を得る。**初回は現在値と同じ**なので、初回だけ `undefined` になる設計ではない。選択項目や表示設定の前回値を参照するときに使える。`import {computedPrevious} from 'ngxtension/computed-previous'`。

[公式資料](https://ngxtension.dev/utilities/signals/computed-previous/)

<a id="api-036"></a>

#### `injectResize`

**使い方**

```ts
readonly size$ = injectResize({ emitInitialResult: true });
// size$ は Observable<ResizeResult>; 値に width / height がある
```

コンポーネントの **ホスト要素** を監視する。`emitInitialResult` は監視開始時のサイズも流し、`debounce` は連続通知を間引く。`toSignal` や `async` パイプで受ける。別の子要素を監視したい場合は `NgxResize` ディレクティブ。`import {injectResize} from 'ngxtension/resize'`。

[公式資料](https://ngxtension.dev/utilities/directives/resize/)

<a id="api-037"></a>

#### `explicitEffect`

**使い方**

```ts
const count = signal(0);
explicitEffect([count], ([value]) => console.log(value));
```

第 1 引数に再実行の依存 Signal を列挙し、第 2 引数でその値を受ける。コールバック内で別の Signal を読んでも依存には加わらない。Angular の注入コンテキストで作り、戻り値の `EffectRef` は必要なら `destroy()` できる。`import {explicitEffect} from 'ngxtension/explicit-effect'`。

[公式資料](https://ngxtension.dev/utilities/signals/explicit-effect/)

<a id="api-038"></a>

#### `injectQueryParams`

**使い方**

```ts
readonly page = injectQueryParams('page'); // URL ?page=2 → page() は '2'
```

現在の URL のクエリパラメータを `Signal<string | null>` で読む。URL 更新に追従し、キーがなければ `null`。数値化したい場合は `injectQueryParams('page', { parse: Number, defaultValue: 1 })` のように指定する。ページ番号や絞り込み条件を URL に持たせるときに使える。`ngxtension/inject-query-params` から import。

[公式資料](https://ngxtension.dev/utilities/injectors/inject-query-params/)

<a id="api-039"></a>

#### `injectParams`

**使い方**

```ts
readonly id = injectParams('id'); // ルート /items/:id → id()
```

現在のルートのパスパラメータを Signal で読む。未設定時は `null`。子ルートまで含むパラメータが必要なら `injectParams.global('id')`。注入コンテキストで作る。詳細画面で対象の ID を読むときに使える。`ngxtension/inject-params` から import。

[公式資料](https://ngxtension.dev/utilities/injectors/inject-params/)

<a id="api-040"></a>

#### `NgxResize` / `ResizeResult`

**使い方**

```ts
@Component({
  template: '<div ngxResize (ngxResize)="onResize($event)"></div>',
  imports: [NgxResize],
})
class ResizeExample {
  onResize(result: ResizeResult) { console.log(result.width, result.height); }
}
```

`NgxResize` は任意の要素へ付けるスタンドアロン・ディレクティブ、`ResizeResult` はイベント値の型。`width` / `height` / `entries` などを持つ。例えば、パネルの高さに応じて一覧の表示量を調整するときに使う。どちらも `ngxtension/resize` から import。

[公式資料](https://ngxtension.dev/utilities/directives/resize/)

<a id="api-041"></a>

#### `injectRouteData`

**使い方**

```ts
readonly title = injectRouteData<string>('title'); // route.data.title を title() で読む
```

ルート定義の `data` や resolver の値を Signal にする。キーがないと `null` になる。ルーティング設定の `data: { title: '一覧' }` と対応させる。画面タイトルや表示設定をルートごとに受け取るときに使える。`ngxtension/inject-route-data` から import。

[公式資料](https://ngxtension.dev/utilities/injectors/inject-route-data/)

<a id="api-042"></a>

#### `mapArray`

**使い方**

```ts
const doubled$ = of([1, 2]).pipe(mapArray(n => n * 2)); // [2, 4]
```

`Observable<T[]>` の配列が流れるたび、各要素に関数を適用して `Observable<R[]>` を返す。実質 `map(items => items.map(...))`。元配列の要素は変更しない。例えば、取得した数値の配列を表示用の値へ変換するときに使う。`ngxtension/map-array` から import。

[公式資料](https://ngxtension.dev/utilities/operators/map-array/)

### 7-3. date-fns

`import {format, parseISO} from 'date-fns'` のように使う。例の `Date` はローカル時刻で生成したものと、末尾 `Z` 付きの UTC 時刻を区別して読む。

<a id="api-043"></a>

#### `format`

**使い方**

```ts
format(new Date(2026, 8, 23, 9, 5), 'yyyy-MM-dd HH:mm') // '2026-09-23 09:05'
```

第 1 引数は `Date`、第 2 引数は表示パターン。`MM` は月、`mm` は分、`dd` は日。表示は実行環境のローカル時刻が基準。一覧の日時表示や、指定書式での出力に使える。

[公式資料](https://date-fns.org/docs/format)

<a id="api-044"></a>

#### `parseISO`

**使い方**

```ts
parseISO('2026-09-23T09:05:00Z') // Date
```

ISO 8601 形式の文字列を `Date` にする。末尾 `Z` は UTC、オフセットなしの日付時刻はローカル時刻として扱う。取得後は `isValid` で妥当性を確認できる。

[公式資料](https://date-fns.org/docs/parseISO)

<a id="api-045"></a>

#### `addDays`

**使い方**

```ts
addDays(new Date(2026, 8, 23), 2) // 9 月 25 日の Date
```

第 2 引数の日数を加算した **新しい Date** を返す。元の Date は変更しない。`-2` なら 2 日前。月末をまたぐ期間計算も暦日単位で処理する。

[公式資料](https://date-fns.org/docs/addDays)

<a id="api-046"></a>

#### `startOfDay`

**使い方**

```ts
startOfDay(new Date(2026, 8, 23, 18, 30)) // 同日の 00:00:00
```

渡した日時を **ローカル時刻** のその日開始に丸めた新しい Date を返す。期間フィルタの開始時刻に利用。UTC の 0 時が必要ならタイムゾーンを別途考慮する。

[公式資料](https://date-fns.org/docs/startOfDay)

<a id="api-047"></a>

#### `subSeconds`

**使い方**

```ts
subSeconds(new Date('2026-09-23T00:01:00Z'), 30) // 30 秒前
```

第 2 引数の秒数を引いた新しい Date を返す。元の Date は変更しない。直近 N 秒の範囲計算に使う。

[公式資料](https://date-fns.org/docs/subSeconds)

<a id="api-048"></a>

#### `isValid`

**使い方**

```ts
isValid(parseISO('2026-09-23')) // true; isValid(new Date('invalid')) // false
```

`Invalid Date` などを除外してから `format` へ渡せる。`isValid` は入力文字列の書式検証そのものではないので、厳密なフォーマット指定には専用の検証も考える。

[公式資料](https://date-fns.org/docs/isValid)

<a id="api-049"></a>

#### `formatDistance`

**使い方**

```ts
formatDistance(new Date(2026, 8, 23), new Date(2026, 8, 26)) // '3 days' など
```

2 つの日付の距離を人間向けの表現にする。**デフォルトでは『前』は付かない**。過去・未来の方向も出すなら `{ addSuffix: true }`、日本語なら `{ locale: ja }` を指定。例えば、更新日時と現在時刻から「何日前」の表示を作るときに使う。

[公式資料](https://date-fns.org/docs/formatDistance)

<a id="api-050"></a>

#### `enGB`（locale）

**使い方**

```ts
format(new Date(2026, 8, 23), 'PPPP', { locale: enGB })
```

`enGB` は関数ではなく英国英語ロケールの定義。`format` / `formatDistance` の `locale` に渡すと月名や日付表現へ反映される。英語表記の日付ラベルを作るときに使える。`import {enGB} from 'date-fns/locale'`。

[公式資料](https://date-fns.org/docs/I18n)

### 7-4. uuid

同じ `uuid` パッケージでも、毎回異なる ID と、入力から再現できる ID では選ぶ関数が異なる。

<a id="api-051"></a>

#### `v4`

**使い方**

```ts
import {v4 as uuidv4} from 'uuid';
const id = uuidv4(); // 毎回異なる UUID v4 文字列
```

乱数を使った UUID。新しいローカル行の一時 ID や要素識別子など、同じ入力から同じ ID にする必要がない場面に使う。生成結果は通常の文字列。

[公式資料](https://github.com/uuidjs/uuid#uuidv4options-buffer-offset)

<a id="api-052"></a>

#### `v5`

**使い方**

```ts
import {v5 as uuidv5} from 'uuid';
const id = uuidv5('order-123', uuidv5.URL);
```

**名前と名前空間 UUID の組**から決定的な UUID を作る。同じ組なら同じ ID。名前が同じでも名前空間が違えば別 ID。用途に応じて固定の名前空間を選ぶ。

[公式資料](https://github.com/uuidjs/uuid#uuidv5name-namespace-buffer-offset)

<a id="api-053"></a>

#### `v1`

**使い方**

```ts
import {v1 as uuidv1} from 'uuid';
const id = uuidv1();
```

時刻情報を含む UUID v1 を作る。時刻とノード識別情報を含むため、公開 ID に採用する前に要件を確認する。文字列の単純ソートで時系列順が保証されるわけではない。

[公式資料](https://github.com/uuidjs/uuid#uuidv1options-buffer-offset)

### 7-5. @ctrl/tinycolor

`import {TinyColor, mostReadable} from '@ctrl/tinycolor'`。加工メソッドごとに元オブジェクトを変更するか確認する。

<a id="api-054"></a>

#### `new TinyColor()`

**使い方**

```ts
const color = new TinyColor('#336699');
```

色文字列や `{ r: 51, g: 102, b: 153 }` などを 1 つの色オブジェクトへ変換する。以降の `.toRgb()` や `.lighten()` はこのインスタンスに対して呼ぶ。グラフやタグの色を加工する出発点になる。`import {TinyColor} from '@ctrl/tinycolor'`。

[公式資料](https://tinycolor.vercel.app/)

<a id="api-055"></a>

#### `.toRgb()`

**使い方**

```ts
new TinyColor('#336699').toRgb() // { r: 51, g: 102, b: 153, a: 1 }
```

赤・緑・青を 0〜255、アルファを 0〜1 の数値で取り出す。チャート API へ数値配列で渡す前の分解に便利。

[公式資料](https://tinycolor.vercel.app/#torgb)

<a id="api-056"></a>

#### `.toHexString()`

**使い方**

```ts
new TinyColor('#336699').toHexString() // '#336699'
```

6 桁の `#rrggbb` 形式へ正規化する。透明度はこの文字列に入らないため、透過色を維持するなら `toHex8String()` や `toRgbString()` を検討。

[公式資料](https://tinycolor.vercel.app/#tohexstring)

<a id="api-057"></a>

#### `.toRgbString()`

**使い方**

```ts
new TinyColor('rgba(51, 102, 153, 0.5)').toRgbString() // rgba(...) 形式
```

CSS で使える `rgb(...)` または、透明度が 1 未満なら `rgba(...)` の文字列にする。グラフやタグの色を CSS へ渡すときに使える。

[公式資料](https://tinycolor.vercel.app/#torgbstring)

<a id="api-058"></a>

#### `.mix()`

**使い方**

```ts
new TinyColor('#ff0000').mix('#0000ff', 50).toHexString() // 2 色を半々に
```

第 1 引数に混ぜる色、第 2 引数に混合割合（0〜100）を指定。新しい `TinyColor` を返す。例えば、グラデーション用の中間色を作るときに使う。

[公式資料](https://tinycolor.vercel.app/#mix)

<a id="api-059"></a>

#### `.lighten()`

**使い方**

```ts
new TinyColor('#336699').lighten(20).toHexString()
```

HSL の明度を 20 ポイント上げた新しい色を返す。引数を省略するとライブラリの既定量。枠線やホバー色の調整に使う。

[公式資料](https://tinycolor.vercel.app/#lighten)

<a id="api-060"></a>

#### `.shade()`

**使い方**

```ts
new TinyColor('#6699cc').shade(20).toHexString()
```

黒を 20% 混ぜた新しい色を返す。`lighten` と違い、単純に HSL の明度だけを操作するわけではない。

[公式資料](https://tinycolor.vercel.app/#shade)

<a id="api-061"></a>

#### `.setAlpha()`

**使い方**

```ts
const color = new TinyColor('#336699');
color.setAlpha(0.5).toRgbString() // rgba(..., 0.5)
```

アルファ値は 0〜1。**このメソッドは元の `TinyColor` 自身を変更して `this` を返す**。元色も残したいなら別インスタンスに適用する。

[公式資料](https://tinycolor.vercel.app/#setalpha)

<a id="api-062"></a>

#### `.toHsl()`

**使い方**

```ts
new TinyColor('#336699').toHsl() // { h, s, l, a }
```

色相 `h` は角度、彩度 `s` と明度 `l` は 0〜1、`a` はアルファ。色相だけを調整する処理の出発点にする。

[公式資料](https://tinycolor.vercel.app/#tohsl)

<a id="api-063"></a>

#### `.isDark()`

**使い方**

```ts
new TinyColor('#222222').isDark() // true
```

ライブラリの明るさ基準で暗い色かを真偽値で返す。文字色の大まかな切り替えには使えるが、アクセシビリティのコントラスト判定には `mostReadable` などで確認する。

[公式資料](https://tinycolor.vercel.app/#isdark)

<a id="api-064"></a>

#### `.getLuminance()`

**使い方**

```ts
new TinyColor('#ffffff').getLuminance() // 1
```

WCAG の相対輝度を 0〜1 で返す。黒は 0、白は 1。候補の読みやすさを計算する基礎値。

[公式資料](https://tinycolor.vercel.app/#getluminance)

<a id="api-065"></a>

#### `.getBrightness()`

**使い方**

```ts
new TinyColor('#ffffff').getBrightness() // 255
```

赤・緑・青の加重平均による明るさを 0〜255 で返す。相対輝度とは尺度と計算方法が異なる。

[公式資料](https://tinycolor.vercel.app/#getbrightness)

<a id="api-066"></a>

#### `mostReadable`

**使い方**

```ts
mostReadable('#336699', ['#ffffff', '#000000'])?.toHexString()
```

背景色と候補色の配列を渡し、コントラストが最も高い候補を `TinyColor` で返す。候補がなければ `null` もあり得る。タグやバッジの背景に合わせて文字色を選ぶときに使える。`import {mostReadable} from '@ctrl/tinycolor'`。

[公式資料](https://tinycolor.vercel.app/#mostreadable)

### 7-6. dompurify

HTML を画面に流す前に使う。`iframe` など許可対象を増やす場合は、許可元のチェックも合わせて行う。

<a id="api-067"></a>

#### `DOMPurify.sanitize`

**使い方**

```ts
const clean = DOMPurify.sanitize('<b>OK</b><script>alert(1)</script>'); // <b>OK</b>
```

信頼できない HTML を表示前にサニタイズする。必要な要素や属性だけを許可する場合は第 2 引数に `{ ADD_TAGS: ['iframe'], ADD_ATTR: ['allow'] }` などを渡す。`iframe` を許可するときは `src` の許可元も検証する。Markdown を変換した HTML や、入力された説明文の表示に使える。`import DOMPurify from 'dompurify'`。

[公式資料](https://github.com/cure53/DOMPurify#can-i-configure-dompurify)

<a id="api-068"></a>

#### `DOMPurify.addHook`

**使い方**

```ts
import DOMPurify from 'dompurify';

DOMPurify.addHook('uponSanitizeElement', (node, event) => {
  if (event.tagName !== 'iframe') return;
  const iframe = node as HTMLIFrameElement;
  try {
    const url = new URL(iframe.getAttribute('src') ?? '', location.href);
    if (url.origin !== location.origin) iframe.removeAttribute('src');
  } catch {
    iframe.removeAttribute('src');
  }
});
const inputHtml = '<iframe src="https://other.example"></iframe>';
const clean = DOMPurify.sanitize(inputHtml, { ADD_TAGS: ['iframe'] });
```

サニタイズ処理の指定段階でコールバックを呼ぶ。例は `iframe` の URL を解釈し、現在のオリジン以外なら `src` を除く。フックは DOMPurify インスタンスへ登録され、繰り返し登録すると重複実行されるため初期化時に一度だけ登録する。不要になれば `removeHook` / `removeAllHooks` を使う。

[公式資料](https://github.com/cure53/DOMPurify#hooks)

### 7-7. semver

`semver/functions/valid` などのサブパスから関数ごとに import する。

<a id="api-069"></a>

#### `valid`

**使い方**

```ts
valid('1.2.3') // '1.2.3'; valid('1.2') // null
```

完全なセマンティックバージョンか確認し、妥当なら正規化した文字列、不正なら `null` を返す。比較前の入力チェックに使う。`import valid from 'semver/functions/valid'`。

[公式資料](https://github.com/npm/node-semver#functions)

<a id="api-070"></a>

#### `gt`

**使い方**

```ts
gt('2.0.0', '1.9.9') // true
```

第 1 バージョンが第 2 より **厳密に新しい** ときだけ `true`。通常の文字列比較では `10.0.0` と `2.0.0` の順を誤るため使わない。入力は `valid` で確かめる。`semver/functions/gt` から import。

[公式資料](https://github.com/npm/node-semver#functions)

<a id="api-071"></a>

#### `gte`

**使い方**

```ts
gte('2.0.0', '2.0.0') // true
```

第 1 バージョンが第 2 と同じか新しければ `true`。最低サポート版数の判定に使う。`semver/functions/gte` から import。

[公式資料](https://github.com/npm/node-semver#functions)

### 7-8. filesize

表示したい単位系と桁数を先に決め、`filesize` のオプションとして渡す。

<a id="api-072"></a>

#### `filesize`

**使い方**

```ts
filesize(1536, { base: 2, round: 1 }) // '1.5 KiB'
```

バイト数を単位付き文字列へ変換する。`base: 2` は 1024 倍ごと、`base: 10` は 1000 倍ごと。`round` で小数桁を指定。ファイルサイズや使用容量の表示に使える。`import {filesize} from 'filesize'`。

[公式資料](https://filesizejs.com)

<a id="api-073"></a>

#### `FilesizeOptions`

**使い方**

```ts
const options: FilesizeOptions = { base: 2, round: 2 };
filesize(2048, options);
```

`FilesizeOptions` は関数ではなく設定オブジェクトの TypeScript 型。`base`、`round`、`symbols` などを型付きで管理できる。画面ごとに桁数や単位表記をそろえるときに使える。`import type {FilesizeOptions} from 'filesize'`。

[公式資料](https://filesizejs.com)

### 7-9. @ngneat/dag

処理の依存関係を段ごとに並べる例。ノードの型を定義し、サービスへ配列を渡す最小構成を示す。

<a id="api-074"></a>

#### `DagManagerService`

**使い方**

```ts
// @Component({ providers: [DagManagerService] }) のコンポーネント内
type Step = DagModelItem & { name: string };
const steps: Step[] = [{ stepId: 1, parentIds: [0], branchPath: 1, name: '開始' }];
const dag = inject(DagManagerService<Step>);
dag.setNewItemsArrayAsDagModel(steps);
const rows = dag.getCurrentDagModel(); // Step[][]
```

`DagManagerService<T>` は DAG の状態を保持するサービス。配列を渡すと段ごとの 2 次元配列になり、`dagModel$` でも更新を購読できる。状態を画面ごとに分けるためコンポーネントの `providers` で提供する。依存関係のある作業や処理をフロー図に配置するときに使える。

[公式資料](https://www.npmjs.com/package/@ngneat/dag)

<a id="api-075"></a>

#### `DagModelItem`

**使い方**

```ts
interface Step extends DagModelItem { name: string }
const steps: Step[] = [{ stepId: 1, parentIds: [0], branchPath: 1, name: '開始' }];
```

`DagModelItem` は関数ではなく DAG ノードの型。**`stepId`、`parentIds`、`branchPath` の 3 項目が必須**。`parentIds` は親の `stepId` の配列で、根ノードでは `[0]` を使うのがライブラリの例。例のように `name` など画面表示用の項目を追加できる。

[公式資料](https://www.npmjs.com/package/@ngneat/dag)

### 7-10. 検索・表示・出力のライブラリ

各ライブラリの使い方を、独立して読める小さな例で示す。

<a id="api-076"></a>

#### `marked.parse`

**使い方**

```ts
import {marked} from 'marked';
const html = marked.parse('# 見出し'); // '<h1>見出し</h1>...'
```

Markdown 文字列を HTML に変換する。説明文やエディタのプレビューを作るときに使える。**出力 HTML はサニタイズされない**ため、信頼できない入力を表示する際は DOMPurify などを通す。`import {marked} from 'marked'`。

[公式資料](https://marked.js.org/)

<a id="api-077"></a>

#### `Diff.diffArrays`

**使い方**

```ts
import * as Diff from 'diff';
Diff.diffArrays(['A', 'B'], ['A', 'C'])
```

変更ブロックの配列を返す。各ブロックの `value` に要素列、`added` / `removed` に追加・削除の印が入る。第 1 引数から第 2 引数への差分なので順番で意味が変わる。オブジェクト要素は既定で参照の等価判定。一覧や設定の変更箇所を表示するときに使える。

[公式資料](https://github.com/kpdecker/jsdiff#api)

<a id="api-078"></a>

#### `lucene.parse`

**使い方**

```ts
const ast = lucene.parse('name:alice AND status:done');
```

Lucene 風の検索式を構文木へ変換する。構文エラーは例外になるため、入力欄の自由入力を処理するなら `try/catch` を置く。例えば、検索欄の文字列からフィールド名や条件を読み取るときに使う。`import * as lucene from 'lucene'` など利用中の import 形式に合わせる。

[公式資料](https://github.com/bripkens/lucene#usage)

<a id="api-079"></a>

#### `isValidQuery`（自作関数の例）

**使い方**

```ts
import * as lucene from 'lucene';

function isValidQuery(query: string): boolean {
  if (!query.trim()) return false;
  try {
    lucene.parse(query);
    return true;
  } catch {
    return false;
  }
}

isValidQuery('name:alice') // true
isValidQuery('') // false
```

検索欄の入力を真偽値で検証したい場合に、`lucene.parse` を包む例。`isValidQuery` はこの例で定義する関数で、空文字や構文エラーには `false` を返す。使用を許可するフィールドや演算子の判定が必要なら、構文解析後に別途確認する。次の公式資料は内部で使う `lucene.parse` の API を示す。

[公式資料](https://github.com/bripkens/lucene#usage)

<a id="api-080"></a>

#### `parseHocon`

**使い方**

```ts
const config = parseHocon('server { port = 8080 }'); // { server: { port: 8080 } }
```

HOCON 文字列を JavaScript の値へ変換する。アプリケーションやサーバーの設定文字列を読むときに使える。入力が壊れている場合に備えて `try/catch` し、変換後の形も確認する。`import parseHocon from 'hocon-parser'`。

[公式資料](https://github.com/yellowblood/hocon-js)

<a id="api-081"></a>

#### `mkConfig`

**使い方**

```ts
import {mkConfig} from 'export-to-csv';
const config = mkConfig({ filename: 'users', useKeysAsHeaders: true });
```

CSV 出力の設定を既定値と合わせて作る。`useKeysAsHeaders` はオブジェクトのキーを見出しに使う。例えば、一覧の列名やファイル名に合わせて設定を作るときに使う。`export-to-csv` から import。

[公式資料](https://github.com/alexcaza/export-to-csv#api)

<a id="api-082"></a>

#### `generateCsv`

**使い方**

```ts
const config = mkConfig({ useKeysAsHeaders: true });
const csv = generateCsv(config)([{ name: 'A', score: 10 }]);
```

1 回目の `(config)` で設定済み関数を作り、2 回目の `(rows)` で行データを CSV 化する。返り値は型上 `CsvOutput` で、単純な `string` と区別される。`config` は `mkConfig` で用意する。

[公式資料](https://github.com/alexcaza/export-to-csv#api)

<a id="api-083"></a>

#### `asString`

**使い方**

```ts
const config = mkConfig({ useKeysAsHeaders: true });
const rows = [{ name: 'A', score: 10 }];
const text: string = asString(generateCsv(config)(rows));
```

`CsvOutput` を普通の文字列として取り出す。クリップボードへのコピーや文字列 API へ渡す前に使う。CSV の区切りや引用符の処理は `generateCsv` 済み。

[公式資料](https://github.com/alexcaza/export-to-csv#api)

<a id="api-084"></a>

#### `download`

**使い方**

```ts
const config = mkConfig({ filename: 'users', useKeysAsHeaders: true });
const rows = [{ name: 'A', score: 10 }];
const csv = generateCsv(config)(rows);
download(config)(csv);
```

ブラウザで CSV のダウンロードを始める。こちらも `(config)(csv)` の 2 段呼び出し。`config.filename` がファイル名に使われる。ユーザー操作に結び付けて呼び、Node.js 上ではファイル書き込み API を使う。

[公式資料](https://github.com/alexcaza/export-to-csv#api)

<a id="api-085"></a>

#### `domtoimage.toBlob`

**使い方**

```ts
const element = document.querySelector<HTMLElement>('#chart');
if (element) {
  const blob = await domtoimage.toBlob(element);
}
```

指定した DOM 要素を画像の `Blob` にする Promise を返す。得た Blob は `URL.createObjectURL(blob)` などで保存できる。外部画像・フォントの読み込みやブラウザ制約で失敗する場合があるため `try/catch`。グラフやカードの表示を画像として保存するときに使える。

[公式資料](https://github.com/tsayen/dom-to-image)

<a id="api-086"></a>

#### `new Convert()`

**使い方**

```ts
const convert = new Convert({ newline: true });
```

ANSI コードから HTML への変換設定を保持するインスタンスを作る。設定を省略して `new Convert()` でもよい。複数のログ行を同じ設定で変換するなら、インスタンスを再利用できる。`import Convert from 'ansi-to-html'`。

[公式資料](https://github.com/rburns/ansi-to-html)

<a id="api-087"></a>

#### `convert.toHtml`

**使い方**

```ts
const convert = new Convert();
const html = convert.toHtml('\u001b[31mERROR\u001b[0m');
```

ANSI の色指定を HTML の `span` などへ変換する。ログを画面に出す場合、ANSI 以外の HTML 入力の扱いも確認してから描画する。色付きのコンソール出力をブラウザで表示するときに使える。

[公式資料](https://github.com/rburns/ansi-to-html)

<a id="api-088"></a>

#### `hasAnsi`

**使い方**

```ts
hasAnsi('\u001b[31mERROR\u001b[0m') // true
```

文字列中に ANSI エスケープシーケンスがあるか判定する。色コードのない普通のログは `false`。変換が必要な行だけ `convert.toHtml` に渡す判定に使える。`import hasAnsi from 'has-ansi'`。

[公式資料](https://github.com/chalk/has-ansi)

<a id="api-089"></a>

#### `stc`

**使い方**

```ts
import stc from 'string-to-color';
const color = stc('support'); // 同じ文字列から同じ色
```

文字列をハッシュして色文字列を返す。項目ごとに安定した色を割り当てたいときに使う。例えば、カテゴリ名からタグの色を決めるときに使う。明るさを調整したい場合は `TinyColor` と組み合わせられる。

[公式資料](https://github.com/Gustu/string-to-color)

<a id="api-090"></a>

#### `select`

**使い方**

```ts
select('button').on('click', () => console.log('clicked'));
```

DOM 要素を D3 のセレクションにし、`.on` でイベントを登録する。CSS セレクタ文字列も渡せる。グラフ内の要素へクリック処理を追加するときなどに使える。Angular 管理下の DOM はライフサイクルに注意。`import {select} from 'd3-selection'`。

[公式資料](https://d3js.org/d3-selection/selecting)

<a id="api-091"></a>

#### `interpolateBasis`

**使い方**

```ts
import {interpolateBasis} from 'd3-interpolate';
const interpolate = interpolateBasis([0, 10, 5]);
const middle = interpolate(0.5);
```

数値列から滑らかな補間関数を作る。返された関数に **0〜1** の位置を渡すと補間値が得られる。例えば、アニメーションの途中の値を滑らかに変化させるときに使う。配列の実際の x 座標を指定する API ではない。

[公式資料](https://d3js.org/d3-interpolate/value)

<a id="api-092"></a>

#### `getBoxToBoxArrow`

**使い方**

```ts
import {getBoxToBoxArrow} from 'curved-arrows';
const [sx, sy, c1x, c1y, c2x, c2y, ex, ey, endAngle] =
  getBoxToBoxArrow(0, 0, 80, 40, 200, 100, 80, 40, { padEnd: 8 });
```

前半 4 値が始点矩形の `x, y, width, height`、次の 4 値が終点矩形。戻り値の始点・制御点 2 組・終点で SVG の `M ... C ...` を描き、角度で矢印の頭を回転する。座標系を同じ基準にそろえる。処理フロー図やカード間の接続線を描くときに使える。

[公式資料](https://github.com/dragonman225/curved-arrows)

<a id="api-093"></a>

#### `parse`

**使い方**

```ts
import {parse} from 'url';
const parsed = parse('https://example.com/files/report.txt');
parsed.protocol // 'https:'; parsed.host // 'example.com'
```

レガシーな Node 互換 `url.parse` を使い、URL 風文字列を分解する。ホスト名やファイルのパスを取り出すときに使える。新規コードで通常の HTTP URL を扱うなら標準の `URL` も検討し、両者の戻り値を混同しない。

[公式資料](https://nodejs.org/api/url.html#urlparseurlstring-parsequerystring-slashesdenotehost)

<a id="api-094"></a>

#### `UrlWithStringQuery`

**使い方**

```ts
import {parse, type UrlWithStringQuery} from 'url';
const parsed: UrlWithStringQuery = parse('https://example.com/files/report.txt');
```

`parse` の戻り値の TypeScript 型で、実行時関数ではない。`protocol`、`host`、`pathname`、`query` などは未設定なら `null` になり得る。値を使う前に存在確認する。

[公式資料](https://nodejs.org/api/url.html#legacy-urlobject)

### 7-11. plotly.js（型定義）

グラフの系列データを型付きで作る例。

<a id="api-095"></a>

#### `PlotData`

**使い方**

```ts
import type {PlotData} from 'plotly.js';
const trace: Partial<PlotData> = { x: [1, 2], y: [3, 4], type: 'scatter' };
```

`PlotData` は系列データの TypeScript 型。`Partial<PlotData>` にすると必要な項目だけを指定できる。`import type` は型の参照に使うため、グラフを描画するには実行時のライブラリも別途読み込む。グラフ API の詳細は Plotly の公式資料を参照。

[公式資料](https://plotly.com/javascript/)
