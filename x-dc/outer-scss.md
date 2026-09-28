# CSS / SCSS 構成リファレンス


- 対象バージョン: `sass` 1.101.0 / `@angular/material` `^22.1.5` / `bootstrap` `^5.3.8` / `primeng` `^22.1.0`
- 抽出日: 2026-10-05

## 0. 読み方

### 表記について

汎用リファレンスとして読めるよう、固有名を含む識別子は次のように置き換えて表記する。実コードを grep するときは実際の名前に読み替える。

| 本資料の表記 | 実体 |
| --- | --- |
| `<common>/` | 共通コンポーネント群が置かれたディレクトリ |
| `<widgets>/` | 埋め込みウィジェット用の第2プロジェクトのルート |
| `app-` | コンポーネントセレクタ／グローバルクラスの接頭辞 |
| `app-icon` / `app-ico-*` | アイコンフォント用のクラス接頭辞 |
| `<palette>` | パレット定義ファイル（`styles/` 直下） |
| `<icons>` | アイコンフォント定義ファイル（`assets/fonts/` 配下） |

ファイル名が汎用語のもの（`style.scss`、`variables.scss`、`customizations/colors.scss` など）はそのまま記載する。

### 3つの層

スタイルは3層に分かれている。どの層を触るかで書き方が変わるので、最初に区別しておく。

| 層 | 場所 | 配信形態 | スコープ |
| --- | --- | --- | --- |
| グローバル | `<common>/styles/` | `global-styles` バンドル1本 | アプリ全体。スコープなし |
| Material 調整 | `<common>/styles/customizations/` | 同上（`style.scss` から `@forward`） | アプリ全体。`mat-*` クラスを直接狙う |
| コンポーネント | 各 `*.component.scss` | コンポーネント単位 | Angular のエミュレート済みカプセル化 |

本プロジェクトでは `ViewEncapsulation.None` は 1 箇所のみ。残り全てのコンポーネントはカプセル化が効いているため、Material の内部 DOM を触るには後述の突破口（`::ng-deep` 等）か、グローバル層での上書きを使う。

---

## 1. 数で見る全体像

### at-rule の出現数

| at-rule | 数 | at-rule | 数 |
| --- | ---: | --- | ---: |
| `@include` | 149 | `@if` | 5 |
| `@use` | 144 | `@supports` | 4 |
| `@forward` | 48 | `@else` | 3 |
| `@mixin` | 32 | `@each` | 2 |
| `@media` | 23 | `@starting-style` | 1 |
| `@import` | 16 | `@font-face` | 1 |
| `@keyframes` | 13 | `@extend` | 1 |
| `@container` | 6 |  |  |

`@use` / `@forward` が主で、`@import` は 16 箇所のみ。うち 11 箇所は `minimal-bootstrap.scss` 内の Bootstrap 読み込み（Bootstrap 5 が `@use` 対応していないため）で、残り 5 箇所が `@import "variables"` の旧式記述。`@extend` は 1 箇所のみで、形の共有は mixin で行う方針が徹底されている。

### カプセル化の突破口

| 記述 | 出現行数 |
| --- | ---: |
| `:host` | 233 |
| `!important` | 221 |
| `::ng-deep` | 176 |
| `:host-context(...)` | 15 |

`::ng-deep` が集中しているのは外部ライブラリの DOM を触るファイル。グラフ描画ラッパー（9 箇所で最多・Plotly）、一覧の詳細パネル（7）、`table.component.scss`（6）、`markdown-editor.component.scss`（4）が上位。

---

## 2. ビルド設定（`angular.json`）

プロジェクトは2つある。

### 本体プロジェクト

```json
"styles": [
  "<common>/assets/fonts/heebo.css",
  "node_modules/ngx-markdown-editor/assets/highlight.js/agate.min.css",
  { "bundleName": "global-styles", "inject": true,
    "input": "<common>/styles/style.scss" }
],
"stylePreprocessorOptions": {
  "includePaths": ["<common>/styles/", "."],
  "sass": { "silenceDeprecations": ["import"] }
},
"inlineStyleLanguage": "scss"
```

3点が構成上の前提になる。

- `includePaths` に `styles/` が入っているため、どの階層の SCSS からでも `@use "variables"` と相対パスなしで書ける（29 箇所がこの形）。`.` も入っているので、プロジェクトルート起点のフルパス記述も通る。
- `silenceDeprecations: ["import"]` で `@import` の非推奨警告を止めている。Bootstrap 5 を取り込むための措置。
- `inlineStyleLanguage: scss` のため、`@Component({styles: ...})` を書く場合も SCSS が使える。ただし本プロジェクトではインライン `styles:` は 0 箇所で、全て外部ファイル（`styleUrls` 298 / `styleUrl` 40）。

### 埋め込みウィジェット用プロジェクト

別エントリ `<widgets>/app/styles/styles.scss` を持ち、テーマ生成だけを本体と共有している。`reboot` / `minimal-bootstrap` / `layout` / `utilities` の読み込みはコメントアウトされ、代わりに必要なユーティリティクラス（`.d-flex`、`.w-100`、`.ms-1`〜`.ms-5` など）を同ファイル内に手書きで複製している。iframe に埋め込む都合でバンドルを削った結果。

Material の読み込みも相対パスで `node_modules` を直接指す形になっている（8 階層遡る）。ビルド設定の `includePaths` が本体と違うため、`@use '@angular/material'` が解決できないことへの回避。

---

## 3. グローバルエントリ `style.scss`

グローバル層の唯一の入口。読み込み順がそのままカスケードの順になる。

```scss
@use '@angular/material' as mat;
@use "sass:meta";
@forward "../assets/fonts/<icons>.scss";         // アイコンフォント @font-face
@forward "reboot";                               // リセット
@forward "minimal-bootstrap";                    // Bootstrap ユーティリティ
@forward "layout";                               // レイアウト用クラス
@forward "utilities";                            // 自前ユーティリティ
@forward "customizations";                       // Material 調整（全 23 ファイル）
@forward "icons";                                // アイコンのサイズ・SVG クラス
@forward "../angular-notifier/styles/core";      // 通知ライブラリ
@forward "../angular-notifier/styles/types/type-default";
@forward "../angular-notifier/styles/themes/theme-material.scss";
@forward "notifications";                        // 通知の自前調整
@forward "printing";                             // 印刷用 @media print
@use "themes";
@use "customizations";

@media (prefers-color-scheme: dark) {
  html { background-color: var(--color-shadow); }
}

:root {
  @include mat.all-component-themes(themes.$theme);
  @include mat.system-level-colors(themes.$theme);
  @include customizations.components(themes.$theme);

  &.dark-mode {
    @include mat.all-component-colors(themes.$dark-theme);
    @include mat.system-level-colors(themes.$dark-theme);
    @include customizations.components(themes.$dark-theme);
  }
}

@include meta.load-css("src/app/shared/custom-styles");
```

要点を4つ。

1. **`:root` に対して全部入りのテーマを流し込んでいる。** `mat.all-component-themes` は Material 全コンポーネントの色・タイポグラフィ・密度トークンを出力する。使っていないコンポーネントのトークンも含まれるため、グローバル CSS は大きい。
2. **ダークは色だけ差し替える。** `&.dark-mode` 側は `all-component-colors`（色のみ）であり、`all-component-themes` を再度呼んでいない。タイポグラフィ・密度はライトで出した定義を共有する。
3. **`html` 要素の `prefers-color-scheme: dark` 対応はこの1行だけ。** 実際のテーマ切替はクラス付与（§6）で行う。この `@media` は、Angular の起動前に素の背景が白く光るのを防ぐためのもの。
4. **末尾に空の拡張点がある。** `meta.load-css` で読み込む `src/app/shared/custom-styles.scss` は現在 0 バイト。配布版で見た目を差し替えるためのフックとして残されている。

### 別系統の拡張点

`ThemeService.loadCustomStyle()` が、設定 `customStyle`（`src/environments/base.ts` の `customStyle?: string`）に URL が入っていれば `<link rel="stylesheet">` を動的に `<head>` へ追加する。ビルドに含まれないため、全スタイルの最後に当たる。

---

## 4. テーマ定義（`themes.scss` / パレット定義）

Material 3 の `define-theme` を使う。

```scss
$theme: mat.define-theme((
  color: (
    theme-type: light,
    primary: $_primary,
    tertiary: $_tertiary,
    use-system-variables: true,
    system-variables-prefix: sys,
  ),
  typography: (
    plain-family: variables.$font-family-base,   // 'Heebo', sans-serif
    brand-family: variables.$font-family-base,
  )
));
```

`$dark-theme` は `theme-type: dark` だけを変えた同形。

### パレット

パレット定義ファイルが `$palettes` マップを持つ。`primary` / `secondary` / `tertiary` / `neutral` / `neutral-variant` / `error` の6系統、各系統に Material 3 のトーン値（`0`〜`100`）が入る。`neutral` だけ `4, 6, 12, 17, 22, 24, 87, 92, 94, 96` の追加トーンを持ち、surface-container 系の段階色に使われる。

`themes.scss` 側で、`primary` と `tertiary` のそれぞれに残り4系統をマージして `define-theme` に渡す形になっている。

```scss
$_rest: (
  secondary: map.get(palette.$palettes, secondary),
  neutral: map.get(palette.$palettes, neutral),
  neutral-variant: map.get(palette.$palettes, neutral-variant),
  error: map.get(palette.$palettes, error),
);
$_primary: map.merge(map.get(palette.$palettes, primary), $_rest);
```

同ファイル先頭には、もう1つ別のパレットマップ（紫寄りの配色）が定義されているが**どこからも参照されていない**。旧配色がコメントアウトされずに残っているだけなので、色を変えるときは `$palettes` 側を触る。

### `use-system-variables: true` の効果

Material のコンポーネントトークンが固定値ではなく `var(--mat-sys-*)` 参照として出力される。これにより、`.dark-mode` クラスを付け替えるだけで全コンポーネントの色が追従する。本プロジェクトの SCSS が `--mat-sys-*` を直接参照しているのは 2 箇所のみ（`--mat-sys-surface-variant`、`--mat-sys-body-large-tracking`）で、通常は次の `--color-*` 層を経由する。

---

## 5. CSS カスタムプロパティ `--color-*`

実装コードが色を参照する際の唯一の正規ルート。`customizations/colors.scss` の `generate-colors($theme)` mixin が、Sass のテーマオブジェクトから CSS 変数へ値を書き出す。

```scss
@mixin generate-colors($theme) {
  --color-primary: #{mat.get-theme-color($theme, primary)};
  --color-on-surface-variant: #{mat.get-theme-color($theme, on-surface-variant)};
  // … Material 3 のロール名をそのまま --color-<role> に写す（約 50 個）

  @if mat.get-theme-type($theme) == light {
    --color-running: #1EA26C;
    --color-failed: #A4020D;
    // …
  } @else {
    --color-running: #50e3c2;
    --color-failed: #FFB4AB;
    // …
  }
}
```

### 使用数順の一覧（上位）

| 変数 | 数 | 性質 |
| --- | ---: | --- |
| `--color-primary` | 153 | Material ロール |
| `--color-outline-variant` | 133 | Material ロール。区切り線のデファクト |
| `--color-on-surface-variant` | 127 | Material ロール。補助テキストのデファクト |
| `--color-on-surface` | 111 | Material ロール。本文 |
| `--color-surface-container-high` | 96 | Material ロール |
| `--color-outline` | 91 | Material ロール。枠線 |
| `--color-surface-container-lowest` | 74 | Material ロール。カード・パネル面 |
| `--color-surface-container-low` | 56 | Material ロール |
| `--color-surface` | 47 | Material ロール |
| `--color-warning` | 32 | 独自（テーマ別の直値） |
| `--color-surface-container` | 32 | Material ロール（後述の上書きあり） |
| `--color-surface-container-highest` | 25 | Material ロール |
| `--color-empty-state` | 25 | 独自。`neutral` の 70/50 トーン |
| `--color-tertiary` | 24 | Material ロール |
| `--color-completed` | 13 | 独自。処理状態色 |

区切り線は `--color-outline-variant`、補助テキストは `--color-on-surface-variant`、面は `--color-surface-container-*` という対応が実質の規約になっている。新しいコンポーネントもこれに合わせるのが安全。

### 処理状態の色

バックエンドが返す処理ステータスをそのまま色名にしたもの。本体色とコンテナ色（背景向けの淡い／濃い色）が対で定義される。

| 状態 | 本体 | コンテナ |
| --- | --- | --- |
| draft | `--color-draft` | `--color-draft-container` |
| pending | `--color-pending` | `--color-pending-container` |
| running | `--color-running` | `--color-running-container` |
| completed | `--color-completed` | `--color-completed-container` |
| failed | `--color-failed` | `--color-failed-container` |
| published | `--color-published` | `--color-published-container` |
| skipped / cached / executed | （なし） | `--color-skipped-container` ほか |

### エイリアスと tint

`colors.scss` の `:root` ブロックで、API が返すステータス名を上の色名に寄せている。

```scss
:root {
  --color-tint-5:  rgba(from var(--color-primary) r g b / 5%);
  --color-tint-8:  rgba(from var(--color-primary) r g b / 8%);
  --color-tint-11: rgba(from var(--color-primary) r g b / 11%);
  --color-tint-12: rgba(from var(--color-primary) r g b / 12%);
  --color-tint-14: rgba(from var(--color-primary) r g b / 14%);

  --color-in_progress: var(--color-running);
  --color-aborted:     var(--color-completed);
  --color-stopped:     var(--color-completed);
  --color-created:     var(--color-draft);
  --color-queued:      var(--color-pending);
  --color-error-container: var(--color-failed-container);
}
```

`--color-in_progress` のようにアンダースコアを含む名前があるのは、API のステータス文字列をテンプレートで直接 `var(--color-{{status}})` に組み立てているため。ステータス名を変えるときは TS 側と対で見る必要がある。

`--color-error-container` は `generate-colors` 内ではコメントアウトされており、Material の `error-container` ではなく `--color-failed-container` を指す。エラー表示の色だけ Material のロールから外れている点に注意。

### 既知の上書き

```scss
--color-surface-container: #{mat.get-theme-color($theme, neutral, 98)}; // Override theme color (white), bug?
```

ライト時に `surface-container` が白く出る問題への回避。コメント付きで残っているため、Material 側の修正後に見直す対象。

---

## 6. ダークモードの切替経路

クラス切替で行う。`prefers-color-scheme` は「システム設定の観測」にしか使わない。

1. `ThemeService`（`shared/services/theme.service.ts`）が `BreakpointObserver.observe(['(prefers-color-scheme: dark)'])` でシステム設定を監視し、`systemThemeChanged` を dispatch。
2. NgRx の `selectThemeMode` が最終的なモード（システム設定・ユーザー設定・`forceTheme` 設定の合成結果）を返す。
3. 変化時に `Renderer2` で `document.documentElement`（= `<html>`）のクラスを入れ替える。

```ts
this.renderer.removeClass(this.document.documentElement, `${prevTheme}-mode`);
this.renderer.addClass(this.document.documentElement, `${theme}-mode`);
```

付くのは `light-mode` / `dark-mode`。SCSS 側が見ているのは `dark-mode` だけで、`light-mode` はどの SCSS からも参照されていない（ライトが既定値）。

4. 併せて `setThemeColors` を dispatch し、`getComputedStyle` で `--color-*` の実値を全部読んで store に入れる。CSS 変数を解釈できない描画ライブラリ（Plotly など）に色を渡すための経路。

### SCSS 側での書き分け

`dark-mode` を直接見ている箇所は全体で 9 行しかない。

| 形 | 使う場所 | 例 |
| --- | --- | --- |
| `@if mat.get-theme-type($theme) == dark` | グローバル層の mixin 内（推奨） | `customizations/table.scss` |
| `:root.dark-mode { … }` | グローバル層で Material の DOM を狙うとき | `customizations/menus.scss`、`form-fields.scss` |
| `:host-context(.dark-mode) { … }` | コンポーネント SCSS（3 箇所） | `login.component.scss` |

原則として、個別に書き分けるのではなく `--color-*` を参照すればテーマは自動で追従する。上の 9 行は、変数で表現しきれなかった例外（枠線の有無、シンタックスハイライトの配色など）。

---

## 7. Sass 変数（`variables.scss`）

196 行。`--color-*` が入る前からある層で、現役の変数と死んでいる変数が混在している。

### 実際に参照されている変数

| 変数 | 参照数 | 内容 |
| --- | ---: | --- |
| `$assets-icons-path` | 46 | SVG アイコンのディレクトリ。パス組み立てに使う |
| `$font-family-base` | 10 | `'Heebo', sans-serif` |
| `$transition-slow` | 9 | `0.6s` |
| `$font-family-monospace` | 8 | SFMono 系のスタック |
| `$nav-bar-height` | 5 | `64px` |
| `$icomoon-font-family` | 5 | アイコンフォント名（定義は `assets/fonts/variables.scss`） |
| `$transition-fast` | 3 | `0.3s` |
| `$side-bar-close-width` | 2 | `64px` |
| `$font-size-small` | 2 | `12px` |
| `$blue-600` / `$blue-700` | 2 / 1 | 旧配色の直参照 |
| `$generic-base-z-index` | 1 | `99999` |

### 死んでいる、または避けるべき変数

- **色系（`$blue-*`、`$white`、`$failed-red`、`$running-green` など 50 以上）**: ほぼ使われていない。`--color-*` に置き換わっている。ただし `customizations/cards.scss` のダーク分岐が `$blue-500`〜`$blue-700` を直接埋めているため、完全には剥がれていない。
- **Material 1 系の旧パレット形式の Sass マップ**（`50`〜`900` + `contrast` の形）: `define-theme` には渡せないので現在は未使用。
- **`// TODO:` 付きの Bootstrap 由来変数（`$spacers`、`$sizes`、`$border-radius`、`$font-weight-*` など）**: 上流が Bootstrap の変数名をコピーしてきた跡。Bootstrap 本体の変数は `minimal-bootstrap.scss` 経由で別に読み込まれるため、これらは二重定義になっている。参照しないこと。
- **`$zindex-*`**: 定義のみ。実際の重なり順は各コンポーネントが直値か `$generic-base-z-index` で書いている。

新しくスタイルを書くとき、`variables.scss` から取るべきなのは実質「フォント・トランジション・アイコンパス・ナビの寸法」だけ。色は `--color-*` を使う。

---

## 8. 共有 mixin（`styles/mixins/`）

| ファイル | mixin | 用途 | 参照数 |
| --- | --- | --- | ---: |
| `link.scss` | `link($link-color)` | 子孫の `a[href]` に色とホバー下線を付ける | 12 |
| `common.scss` | `vertical-align`、`counter-label`、`counter-title`、`card-sub-title`、`recent-title`、`card-header-tag` | 数値表示とカード見出しの体裁 | 11 |
| `empty.scss` | `table-empty-message`、`empty-card-mixin` | 「データなし」表示 | 7 |
| `icon.scss` | `icon($url)` | SVG を `background-image` として貼る | （`icons.scss` 内で 46 回） |
| `system-tag.scss` | `app-system-tag`、`system-tag`、`shrinkable-tags` | システムタグのチップ表現 | 1 |
| `wizard-template.scss` | `common-leaf`、`choose-leaf` | ウィザードのカード下線アニメーション | 1 |

`icon.scss` の `icon($url)` は相対パスを4階層遡る前提で書かれている。

```scss
@mixin icon($url) {
  background-image: url('../../../../#{$url}');
  background-repeat: no-repeat;
  background-position-x: center;
  background-position-y: center;
  background-size: contain;
  display: inline-block;
}
```

そのため、`styles/icons.scss` 以外のファイルから呼ぶとパスが壊れる。SVG アイコンを増やすときは `styles/icons.scss` に追記するのが前提になっている。

---

## 9. Material の調整（`styles/customizations/`）

23 ファイル。`_index.scss` が全部を `@forward` し、テーマ依存のものだけ `components($theme)` mixin に集約する。

```scss
@mixin components($theme) {
  @include colors.generate-colors($theme);
  @include table.table($theme);
  @include buttons.buttons($theme);
  @include cards.cards($theme);
  @include dialogs.dialogs($theme);
  @include sidenav.drawer($theme);
}
```

つまり各ファイルは2つの部分に分かれる。

- **トップレベルに書かれた部分**: テーマに依存しない。ライト／ダーク共通で1回だけ出力される。
- **`@mixin xxx($theme)` の中**: `components($theme)` 経由で、ライトと `.dark-mode` の2回出力される。

| ファイル | 対象 | テーマ依存 mixin |
| --- | --- | --- |
| `colors.scss` | `--color-*` の生成 | `generate-colors($theme)` |
| `buttons.scss` | ボタン高さ（36/24/20px の3段）、`.icon-only`、アイコン寸法 | `buttons($theme)` |
| `form-fields.scss` | フォームフィールド高さ 36px 固定、ラベル浮き上がり、select/autocomplete パネル | — |
| `menus.scss` | メニューパネルの枠線・幅・項目高さ 40px | — |
| `dialogs.scss` | ダイアログサイズクラス（`.dialog-sm`〜`.dialog-xxl`、`.full-screen`） | `dialogs($theme)` |
| `table.scss` | `--row-*` 変数群 | `table($theme)` |
| `tooltip.scss` | ツールチップの文字サイズ、`.break-line` / `.scrollable` / `.validation` | — |
| `cards.scss` | `--card-tab1-bg` ほか | `cards($theme)` |
| `sidenav.scss` | ドロワーの背景・スクリム | `drawer($theme)` |
| `expansion-panel.scss` | 影を消す、`.grouped-selection` の詰めたヘッダ | — |
| `tabs.scss` | `mat.tabs-overrides` で区切り線とリップルを消す | — |
| `scrollbar.scss` | WebKit スクロールバー + Firefox フォールバック | — |
| `icons.scss` | `.mat-icon` のサイズ段階（`xs`/`sm`/`lg`/`xl`/`xxxl`） | — |
| `slide-toggle.scss` / `slider.scss` | 既定より小さい寸法に揃える | — |
| `divider.scss` / `links.scss` / `autocompletes.scss` / `markdown.scss` | 単機能の調整 | — |
| `ace.scss` / `angular-split.scss` | 外部ライブラリの配色をテーマに合わせる | — |

### 上書きの3手段

Material のトークンを上書きする方法が混在している。新規に書くときは上から選ぶ。

**1. 公式の override mixin（6 箇所）**

```scss
.mat-mdc-button-base {
  @include mat.button-overrides((
    filled-container-height: 36px,
    outlined-container-height: 36px,
    text-container-height: 36px,
  ));
}
```

使用実績は `button-overrides`（4）、`tabs-overrides`、`icon-button-overrides`、`chips-overrides`、`badge-overrides`。トークン名の誤りをコンパイル時に検出できるため、これが第一候補。

**2. CSS 変数の直接宣言（最多）**

```scss
.mat-mdc-form-field {
  --mat-form-field-container-height: 36px;
  --mat-form-field-outlined-outline-color: var(--color-outline-variant);
}
```

override mixin が用意されていないコンポーネント、または `var(--color-*)` を値として渡したいときに使う。トークン名を間違えても無言で効かないだけなので、名前は Material のドキュメントか DevTools で確認する。

**3. プロパティの直接指定**

```scss
body mat-expansion-panel.mat-expansion-panel {
  box-shadow: none !important;
}
```

トークンが存在しない事柄（上の例では影の除去）に対する最後の手段。`body` を付けて詳細度を稼ぐ、`!important` を付けるといった記述が必要になる時点で、Material のバージョンアップで壊れやすい箇所だと分かる。

### `mat.get-theme-color` の使い方

68 箇所。2つの呼び方がある。

```scss
mat.get-theme-color($theme, primary)            // ロール名で引く
mat.get-theme-color($theme, neutral, 98)        // パレットとトーンで引く
```

前者はテーマが用意したロールを取るので意味が安定する。後者はパレットの生の色を取るため、ロールに無い中間色が必要なとき（`--color-empty-state`、`--color-diff-line-*` など）に使われている。

---

## 10. ユーティリティクラス

2系統が共存する。

### 自前（`utilities.scss`）

| クラス | 内容 |
| --- | --- |
| `.pointer` / `.grab` / `.cursor-default` | カーソル |
| `.pointer-events-none` | `pointer-events: none !important` |
| `.noselect` | `user-select: none` |
| `.resize-none` | `resize: none !important` |
| `.hidden` | `visibility: hidden` + `opacity: 0`（領域は残す） |
| `.ellipsis` | 1行省略。`.inline` 併用で `inline-block` |
| `.font-weight-bold` | `font-weight: 700` |

同ファイルで `--font-family-base` / `--font-family-monospace` を `:root` に出している。Sass 変数と CSS 変数の両方で同じフォントスタックが定義されているため、どちらを参照しても同じ値になる。

### Bootstrap 5（`minimal-bootstrap.scss`）

```scss
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";
@import "bootstrap/scss/variables-dark";
@import "bootstrap/scss/maps";
@import "bootstrap/scss/mixins";
@import "bootstrap/scss/utilities";
@import "bootstrap/scss/helpers";
@import "bootstrap/scss/utilities/api";
```

コンポーネント（`.btn`、`.card`、`.modal` など）は読み込まず、ユーティリティ API とヘルパーだけを使う。グリッドの `.row` / `.col-*` も Bootstrap の定義ではなくヘルパー経由で部分的に効く形。

テンプレートでの使用数（上位）:

| クラス | 数 | クラス | 数 |
| --- | ---: | --- | ---: |
| `d-flex` | 211 | `flex-column` | 18 |
| `container` | 185 | `mt-2` | 16 |
| `align-items-center` | 86 | `flex-grow-1` | 14 |
| `w-100` | 66 | `mb-3` | 13 |
| `h-100` | 36 | `my-2` | 12 |
| `justify-content-center` | 30 | `mx-auto` / `mb-2` | 11 / 11 |
| `row` | 24 | `overflow-hidden` | 10 |
| `justify-content-between` | 22 | `gap-2` / `gap-3` | 7 / 18 |

`container` の 185 はほとんどが Bootstrap の `.container` ではなく、`.generic-container` のような自前クラス名との部分一致を含む集計。純粋な Bootstrap の `.container` 使用はこれより少ない。

### レイアウト（`styles/layout.scss`）

グローバルなレイアウトクラスを定義する。`.app-card-list-layout`（カードのグリッド）、`.app-entity-page-header`、`.page-header`、`.d-flex-center`、`.flex-middle`、`.stick-to-corner`。接頭辞はこのアプリのコンポーネントセレクタと同じものを使っている。

これとは別に `<common>/layout/layout.scss` が**寸法の Sass 変数だけ**を持つ（クラスは定義しない）。

| 役割 | 値 |
| --- | --- |
| 上部バー | 64px |
| 一覧ヘッダ | 80px（パディング 24px × 2 + 32px） |
| 詳細画面ヘッダ | 56px |
| フッタ | 100px |
| 進捗バー | 46px |
| 詳細画面のタブ | 65px |
| グラフ領域 | 364px |

`calc(100% - #{layout.$top-bar-height})` の形で高さを差し引く計算に使う。各コンポーネントから `@use "../layout/layout"` のように相対パスで読む（相対の深さがファイルごとに違うため、同じファイルを指す `@use` が 5 種類の書き方で現れる）。

---

## 11. アイコン

3系統が併存する。テンプレート側の見分け方も併記する。

### アイコンフォント `app-ico-*`

`assets/fonts/` 配下のアイコンフォント定義ファイル + `assets/fonts/variables.scss`。TTF を `@font-face` で読み、311 個のグリフを Sass 変数（コードポイント文字列）として持つ。

```scss
// variables.scss
$icomoon-font-family: "<フォント名>" !default;
$app-ico-company: string.unquote('"\\ea35"');

// アイコンフォント定義ファイル
.app-ico-company {
  &:before { content: variables.$app-ico-company; }
}
```

テンプレートでは `<mat-icon class="app-icon app-ico-company">` のように、`mat-icon` のクラスとして使われる（`mat-icon` のリガチャ機能ではなく `:before` で描く）。

SCSS から任意の要素にアイコンを埋めることもできる。`customizations/markdown.scss` が、Markdown レンダラが出す `.fa-check-square` を自前フォントに差し替えている例。

```scss
.fa-check-square {
  font-family: variables.$icomoon-font-family;
  &::before { content: variables.$app-ico-sqr-ok; }
}
```

### SVG 背景 `i-*`

`styles/icons.scss` が `mixins/icon.scss` の `icon($url)` を使って 46 個のクラスを定義する。多色のロゴ（ML フレームワーク、クラウドプロバイダ、外部サービス）や、フォント化すると潰れるアイコンがこちら。

```scss
.i-SomeLogo {
  @include icon.icon('#{variables.$assets-icons-path}/some-logo.svg');
}
```

処理状態の `i-*`（`.i-in_progress`、`.i-completed`、`.i-failed`、`.i-published`、`.i-created`）は `.bw`（白黒）、`.dark`、`.notify` といった修飾クラスで別 SVG に差し替わる。テーマ別に SVG を持つもの（`.i-no-plots-dark` など）もあり、ここは CSS 変数で解決できていない領域。

### サイズ段階

`.app-icon` / `.icon` と `.mat-icon` で**別々に**サイズ段階が定義されている（`styles/icons.scss` と `customizations/icons.scss`）。段階が一致していない点に注意。

| クラス | `.app-icon` / `.icon` | `.mat-icon` |
| --- | ---: | ---: |
| `.xs` | 10px | 14px |
| `.msm` | 13px | — |
| `.sm` | 16px | 16px |
| `.sm-md` | 20px | — |
| `.lm` | 24px | — |
| `.md` | 28px | — |
| `.lg` | 32px | 32px |
| `.l-40` | 40px | — |
| `.xl` | 48px | 48px |
| `.xxl` | 64px | — |
| `.xxxl` | 88px | 88px |
| （既定） | 20px | 20px |

`width` / `height` / `font-size` を同じ値で揃える形。`.mat-icon` 側は `:before { line-height: … }` も併せて指定する（アイコンフォントを `:before` で描くため）。

`.app-icon.white` は `filter: brightness(0) contrast(1) grayscale(1) invert(1)` で強制的に白にする。SVG 背景アイコンを反転させるための措置で、フォントアイコンには `color` を使う。

---

## 12. コンポーネント SCSS の書き方

`features/` 配下に新しく追加された機能フォルダが、本プロジェクトでの規範になっている。

```scss
// feature-page.component.scss
@use '../../styles/<機能>.mixins' as f;

.feature-page {
  display: flex;
  flex-direction: column;
  gap: 24px;
  padding: 24px;

  &__lead {
    margin: 4px 0 0;
    color: var(--color-on-surface-variant);
  }

  &__hint {
    @include f.hint;
  }

  &__error {
    margin: 0;
    padding: 12px;
    border-radius: 4px;
    // 色だけで失敗を伝えないよう、role="alert" と併用している。
    background: var(--color-error-container);
    color: var(--color-on-error-container);
  }
}
```

```scss
// run-summary.component.scss
@use '../../styles/<機能>.mixins' as f;

:host {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.run-summary {
  &__facts { @include f.facts; }
  &__controls { @include f.controls; }
}
```

読み取れる規約は5点。

1. **`:host` に `display` を明示する。** Angular のコンポーネントホストは既定で `display: inline` のため、Flex / Grid コンテナにするには明示が必要。`:host` の 233 箇所の多くがこれ。
2. **BEM（`&__`）でクラスを組む。** `&__` が 70 箇所、`&--` が 7 箇所。ただし状態は `&--` ではなく `&.` 形式のクラス（454 箇所）で表現するのが実態。`&.active`、`&.selected`、`&.sm` の形。
3. **色は書かない。** 参照するのは `--color-*` だけ。
4. **画面内で重複する形は機能フォルダの mixin へ。** `features/<機能>/styles/_<機能>.mixins.scss` が `hint` / `controls` / `facts` / `data-table` を持つ。現在そうした mixin ファイルは2つあり、内容が重複したまま意図的に残されている（ファイル先頭のコメントにその判断が書かれている）。
5. **ファイル名の接頭辞 `_`。** mixin だけを持つ partial は `_<name>.mixins.scss`、参照側は `@use '../../styles/<name>.mixins'`（`_` と拡張子を省く）。

### 共通層側の実態

上流由来のコンポーネント SCSS は上の規約に従っていない。BEM ではなく意味クラスの入れ子、`px` の直書き、`::ng-deep` と `!important` の併用が一般的。既存ファイルを直すときは周囲の書き方に合わせ、新規追加は `features/` 側の規約に合わせるのが衝突を避ける。

---

## 13. カプセル化の突破口

### `::ng-deep`（176 箇所）

コンポーネント SCSS から子コンポーネントや外部ライブラリの内部 DOM に届かせる。Angular では非推奨だが代替がまだない。

用途はほぼ以下に限られる。

- グラフ描画ライブラリ（Plotly）の内部 DOM。グラフラッパーの SCSS が 9 箇所で最多
- Material のオーバーレイに出る DOM（`cdk-overlay-*` はコンポーネントの外に出るため、そもそもカプセル化が届かない）
- `ngx-markdown-editor`、`angular-split`、PrimeNG の DataTable

新規に書くなら、まずグローバル層（`customizations/`）での上書きを検討する。オーバーレイ系は特に、`::ng-deep` ではなくパネルに付けたクラス（`panelClass`、`matMenuClass` など）をグローバルで狙うほうが安定する。

### `:host-context(...)`（15 箇所）

祖先要素のクラスに応じて自分の見た目を変える。

| セレクタ | 数 |
| --- | ---: |
| `:host-context(.dark-mode)` | 3 |
| `:host-context(.normal-size)` | 2 |
| `:host-context(.dark-theme)` | 2 |
| その他（`.mobile`、`.selected`、`.single-selection` など） | 各 1 |

`.dark-theme` と `.dark-mode` が混在している。`.dark-theme` は埋め込みウィジェット側で使われる別系統のクラスで、本体アプリでは付かない。

### `!important`（221 箇所）

集中しているのは4系統。

- 埋め込みウィジェットの `styles.scss`（16）: ユーティリティクラスの手書き複製
- グラフラッパー（15）/ `markdown-editor.component.scss`（13）: 外部ライブラリのインラインスタイル上書き
- `_printing.scss`（10）: `@media print` 内での強制上書き（妥当な用法）
- `styles/layout.scss`（8）: ユーティリティクラスの性質上（`.d-flex-center` など）

---

## 14. 外部ライブラリの取り込み

| ライブラリ | 取り込み方 | 調整ファイル |
| --- | --- | --- |
| Bootstrap 5 | `@import` でユーティリティのみ | `minimal-bootstrap.scss` |
| PrimeNG 22 | TS の design token preset | `styles/prime.preset.ts` |
| ace-builds | グローバル SCSS で `.ace_*` を直接狙う | `customizations/ace.scss` |
| angular-split | 同（`as-split` 要素） | `customizations/angular-split.scss` |
| angular-notifier | ライブラリ同梱 SCSS を `@forward` + 自前調整 | `notifications.scss` |
| ngx-markdown-editor | `angular.json` の `styles` で highlight.js の CSS を直読み | コンポーネント SCSS 内で `::ng-deep` |
| Plotly | CSS 変数を読めないため、`ThemeService.getAllThemeColors()` の値を TS 経由で渡す | — |

### PrimeNG の扱いが他と違う点

PrimeNG 22 は SCSS ではなく TS の design token で設定する。`styles/prime.preset.ts` が `definePreset` でトークンを宣言し、`app.config.ts` の `providePrimeNG({ theme: { preset } })` で適用する。

```ts
export const preset = definePreset({}, {
  components: {
    datatable: {
      bodyCell: {
        borderColor: 'var(--row-border-color)',
        selectedBorderColor: 'var(--row-border-color)',
      },
      css: () => `
        th.resize-enabled .p-datatable-column-resizer:hover {
          background: color-mix(in srgb, var(--color-primary), transparent 80%);
        }
      `
    } as DataTableDesignTokens,
  }
});
```

トークンの値に `var(--color-*)` / `var(--row-*)` をそのまま文字列で渡せるため、PrimeNG もアプリのテーマに追従する。`css: () => \`…\`` で生の CSS も注入できる。

**PrimeNG のスタイルを探すときは SCSS を grep しても出てこない。** `prime.preset.ts` を見る。ただし `.p-datatable` に対するコンポーネント SCSS 側の上書きも別に存在するため、両方を確認する必要がある。

### `--row-*` 変数（テーブル行の色）

`customizations/table.scss` が定義し、`prime.preset.ts` と自前のテーブルコンポーネント（`app-table`）の両方から参照される。

| 変数群 | 状態 |
| --- | --- |
| `--row-header-color` / `--row-header-hover-color` | ヘッダ行 |
| `--row-default-color` / `--row-default-text-color` / `--row-default-hover-color` | 通常行 |
| `--row-selection-color` / `--row-selection-hover-color` / `--row-selection-border-color` | 選択モード中の未選択行 |
| `--row-selected-color` / `--row-selected-text-color` / `--row-selected-hover-color` | 選択済み行 |
| `--row-border-color` | 罫線 |

ホバー色は `color-mix` で機械的に作る。

```scss
--row-default-hover-color: color-mix(in srgb, var(--color-surface-container-lowest), var(--color-on-surface) 4%);
```

ダーク時は `table($theme)` mixin 内で混合比を変える（4% → 10%、選択済みのホバーは 20% → 40%）。暗い面では同じ比率だと差が見えないため。

---

## 15. 使われている CSS 機能

| 機能 | 出現数 | 主な用途 |
| --- | ---: | --- |
| `calc()` | 99 | 高さから固定ヘッダ分を引く |
| `gap` | 149 | Flex / Grid の間隔。`margin` より優勢 |
| `grid-template-*` | 72 | カード一覧、定義リスト（`max-content 1fr`） |
| `color-mix()` | 57 | ホバー色・淡色の機械的生成 |
| `max()` | 15 | `padding-left: max(12px, var(--mat-form-field-outlined-container-shape))` |
| `rgba(from …)` / `rgb(from …)` | 14 / 2 | 既存色から透明度だけ変える |
| `@keyframes` | 13 | `fade-in`（5 箇所）、`spin`、`rotation`、`dot`、`expand`、`glow-grow`、`show-step` / `hide-step`、`slide-fade` |
| `:has()` | 7 | `.mat-mdc-form-field:has(.mat-mdc-floating-label)` のように子の有無で親を分岐 |
| `@container` | 6 | フォームの折返し |
| `@supports` | 4 | Firefox 判定（`-moz-appearance`）、`::-webkit-scrollbar` の有無 |
| `@starting-style` | 1 | 出現アニメーションの初期値 |

使われていない: `clamp()`、`light-dark()`、`oklch()`、`aspect-ratio`、`:is()`、`:where()`、`@scope`、`subgrid`、`dvh` / `svh`、`text-wrap`、`field-sizing`、`accent-color`、`backdrop-filter`、`content-visibility`、`scroll-snap`、`view-transition`、anchor positioning。

`color-mix` と `rgb(from …)` を既に前提にしているため、ブラウザサポートの下限はかなり新しい。`.browserslistrc` も「各ブラウザの最新 2 バージョン + Firefox ESR」のみを対象にしている。

```
last 2 Chrome versions
last 2 Firefox version
last 2 Edge major versions
last 2 Safari major versions
last 2 iOS major versions
Firefox ESR
```

新規に書くときも同世代の機能は使える。

### `@container` の使い方

コンテナを宣言する側と問い合わせる側が別ファイルに分かれる。

```scss
// 親コンポーネントの SCSS
.some-wrapper {
  container-type: inline-size;
  container-name: form-container;
}

// 子コンポーネントの SCSS
@container form-container (max-width: 700px) {
  // …
}
```

`container-name` が文字列で結ばれるだけなので、grep で両側を確認しないと壊れたことに気づけない。現在のコンテナ名は `form-container` と `section` の2つ。

---

## 16. 単位と色の直書き

| 単位 | 出現数 |
| --- | ---: |
| `px` | 3,559 |
| `%` | 745 |
| `rem` | 31 |
| `em` | 12 |

`px` が圧倒的で、`rem` は Bootstrap 由来の値（`0.25rem` など）がほとんど。`html` の `font-size` は `reboot.scss` で `14px` に固定されているため、`rem` を使っても `16px` 基準にはならない。寸法は `px` で書くのが既存に合う。

```scss
// reboot.scss
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  font-family: variables.$font-family-base;
  font-size: 14px;
  overflow: hidden;
}
```

`overflow: hidden` が `html, body` に付いているため、ページ全体のスクロールは発生しない。スクロールは常に内側のコンテナが持つ。

### 色の直書き

グローバル層（`styles/`）を除いたコンポーネント SCSS で、生の hex 色は 53 箇所・17 ファイル。全体から見れば少ないが、ダークテーマでコントラストが崩れる箇所の候補になる。`features/` 配下（本プロジェクトで追加された部分）では `dashboard.component.scss` の `#a7b2d8` が唯一の直書きで、新しく追加した機能フォルダには 1 箇所もない。

### `reboot.scss` の `outline: none !important`

```scss
* { outline: none !important; }
```

全要素のフォーカスリングを消している。上流由来の記述で、キーボード操作時のフォーカス可視性が失われる。Material のコンポーネントは独自のフォーカス表現（state layer）を持つので実害が出にくいが、素の `<button>` や `<a>`、自前のクリック可能要素ではフォーカス位置が分からなくなる。アクセシビリティを扱う場合はここを起点に見る。

---

## 17. 変更時に見る場所

| やりたいこと | 触る場所 |
| --- | --- |
| アプリ全体の配色を変える | パレット定義ファイルの `$palettes` |
| 色の意味を増やす（状態色など） | `customizations/colors.scss` の `generate-colors`。ライト／ダーク両方 |
| Material コンポーネントの寸法・色を変える | `customizations/` の該当ファイル。`mat.*-overrides` を優先 |
| ダイアログのサイズを選ぶ | `customizations/dialogs.scss` のクラスを `panelClass` に渡す |
| テーブル行の色を変える | `customizations/table.scss` の `--row-*` |
| PrimeNG のコンポーネントを調整 | `styles/prime.preset.ts` |
| アイコンを追加（単色） | アイコンフォントの再生成（TTF と `assets/fonts/variables.scss`） |
| アイコンを追加（多色・SVG） | `styles/icons.scss` に `@include icon.icon(...)` |
| 全画面共通のレイアウトクラスを足す | `styles/layout.scss` |
| 画面内で形を共有する | 機能フォルダに `_<name>.mixins.scss` |
| 配布向けに見た目を差し替える | `src/app/shared/custom-styles.scss`（現在空）、または設定 `customStyle` の URL |

---

## 付録: 抽出に使ったコマンド

```bash
# CSS 変数の使用数
grep -rhoE 'var\(--[a-zA-Z0-9-]+' --include='*.scss' --include='*.html' --include='*.ts' src/ \
  | sed 's/var(//' | sort | uniq -c | sort -rn

# at-rule の出現数
grep -rhoE '@[a-z-]+' --include='*.scss' src/ | sort | uniq -c | sort -rn

# Material の override mixin
grep -rhoE 'mat\.[a-z-]+-overrides' --include='*.scss' src/ | sort | uniq -c | sort -rn

# Sass 変数の参照数
grep -rhoE 'variables\.\$[a-z0-9-]+' --include='*.scss' src/ | sort | uniq -c | sort -rn

# カプセル化の突破口の所在
grep -rc '::ng-deep' --include='*.scss' src/ | grep -v ':0' | sort -t: -k2 -rn

# コンポーネント SCSS 内の生 hex 色
grep -rhoE '#[0-9a-fA-F]{3,8}\b' --include='*.scss' src/app \
  --exclude-dir=styles --exclude-dir=assets | wc -l

# Bootstrap ユーティリティクラスの使用数
grep -rhoE 'class="[^"]*"' --include='*.html' src/ \
  | grep -oE '\b(d-flex|align-items-[a-z]+|m[trblxy]?-[0-9a-z]+|gap-[0-9])\b' \
  | sort | uniq -c | sort -rn
```

`class` 属性の集計は部分一致のため、`container` が `.generic-container` を拾うなどの誤検出が混ざる。本資料ではそれが起きている箇所に注記を入れてある。
