# Angular Material 要素リファレンス

- 対象バージョン: `@angular/material` / `@angular/cdk` / `@angular/core` いずれも `^22.1.5`

## 0. 読み方

Angular Material の識別子は3種類の形で現れる。混同しやすいので最初に区別しておく。

| 形 | 例 | 正体 |
| --- | --- | --- |
| 要素セレクタ | `<mat-form-field>` | コンポーネント。DOM にその要素が出る |
| 属性セレクタ | `<button mat-icon-button>` `<input matInput>` | ディレクティブ。既存要素に機能と見た目を付与する |
| `exportAs` 名 | `<mat-menu #menu="matMenu">` | インスタンスをテンプレート変数として取り出すための名前 |

同じ名前が要素と属性の両方で使えるものもある（`mat-dialog-actions`、`mat-list-item` など）。本資料では節ごとに両方を併記する。

---

## 1. 要素タグ一覧（出現数順）

出現数は開始タグの数。

| タグ | 数 | タグ | 数 |
| --- | ---: | --- | ---: |
| `mat-icon` | 387 | `mat-radio-group` | 4 |
| `mat-form-field` | 132 | `mat-panel-title` | 4 |
| `mat-label` | 77 | `mat-list-item` | 4 |
| `mat-error` | 64 | `mat-datepicker-toggle` | 3 |
| `mat-option` | 44 | `mat-chip-listbox` | 3 |
| `mat-menu` | 30 | `mat-tree` | 2 |
| `mat-checkbox` | 29 | `mat-select-trigger` | 2 |
| `mat-select` | 19 | `mat-progress-spinner` | 2 |
| `mat-spinner` | 18 | `mat-nav-list` | 2 |
| `mat-slide-toggle` | 16 | `mat-datepicker` | 2 |
| `mat-tab` | 15 | `mat-chip-option` | 2 |
| `mat-expansion-panel` | 14 | `mat-tab-nav-panel` | 1 |
| `mat-expansion-panel-header` | 14 | `mat-stepper` | 1 |
| `mat-radio-button` | 12 | `mat-sidenav-container` | 1 |
| `mat-dialog-actions` | 12 | `mat-sidenav` | 1 |
| `mat-drawer-container` | 11 | `mat-sidenav-content` | 1 |
| `mat-drawer` | 11 | `mat-selection-list` | 1 |
| `mat-drawer-content` | 11 | `mat-list-option` | 1 |
| `mat-slider` | 9 | `mat-list` | 1 |
| `mat-tab-group` | 8 | `mat-progress-bar` | 1 |
| `mat-divider` | 8 | `mat-date-range-input` | 1 |
| `mat-autocomplete` | 8 | `mat-date-range-picker` | 1 |
| `mat-tree-node` | 5 | `mat-chip-grid` | 1 |
| `mat-step` | 5 | `mat-chip-row` | 1 |
|  |  | `mat-button-toggle-group` | 1 |
|  |  | `mat-button-toggle` | 1 |

計 49 種類。インポートされているモジュールは 26 個（`@angular/material/core` を含む）。

---

## 2. アイコン

### `mat-icon`

本プロジェクトで最も多用される要素（387 箇所）。アイコンフォントのリガチャ、または登録済み SVG を表示する。

```html
<mat-icon>search</mat-icon>                    <!-- フォントのリガチャ名 -->
<mat-icon [svgIcon]="'custom-name'"></mat-icon> <!-- MatIconRegistry に登録した SVG -->
<mat-icon fontSet="my-icons" fontIcon="i-star"></mat-icon> <!-- 独自アイコンフォント -->
```

本体は `<span>`/`<svg>` を内側に持つ正方形のインライン要素。`font-size` ではなくホスト要素のサイズ（既定 24px）で大きさが決まるので、拡大縮小は CSS で `font-size` / `width` / `height` を揃えて指定する。

### `matBadge` 系（属性）

アイコンやボタンの角に数値・点を重ねるディレクティブ。

| 識別子 | 役割 |
| --- | --- |
| `matBadge` | バッジに表示する内容 |
| `matBadgePosition` | `above\|below` と `before\|after` の組み合わせで表示位置 |
| `matBadgeSize` | `small\|medium\|large` |

---

## 3. ボタン

本プロジェクトでは属性セレクタの形で 4 種類が使われている。

| 識別子 | 数 | 見た目 |
| --- | ---: | --- |
| `mat-icon-button` | 118 | 円形・背景なし。中身は `mat-icon` 1 個を想定 |
| `mat-flat-button` | 54 | 塗りつぶし。影なし。主要アクション向け |
| `mat-stroked-button` | 40 | 枠線のみ。副次アクション向け |
| `mat-button` | 29 | 背景・枠線なしのテキストボタン |

```html
<button mat-flat-button color="primary" (click)="save()">保存</button>
<button mat-icon-button aria-label="閉じる"><mat-icon>close</mat-icon></button>
```

いずれも `<button>` と `<a>` の両方に付けられる。`<a>` に付けた場合はリンクとして振る舞いつつボタンの見た目になる。新しめのバージョンでは `matButton="filled"` のように外観を入力で渡す統合 API も用意されているが、本プロジェクトは従来のセレクタで統一されている。

### `mat-button-toggle-group` / `mat-button-toggle`

ボタンを横に連結してラジオ（単一選択）またはチェックボックス（複数選択）として機能させるグループ。見た目はボタン、意味は選択コントロール。

```html
<mat-button-toggle-group [(value)]="mode">
  <mat-button-toggle value="list">リスト</mat-button-toggle>
  <mat-button-toggle value="grid">グリッド</mat-button-toggle>
</mat-button-toggle-group>
```

`multiple` を付けると複数選択になり、値は配列になる。

---

## 4. フォームフィールド

### `mat-form-field`

入力コントロールを囲んで、ラベル・枠線・下線・エラー表示・サフィックスの配置を一手に担うコンテナ。132 箇所。これ単体では何も入力できず、必ず中に「フォームフィールドコントロール」を1つ置く必要がある。

```html
<mat-form-field appearance="outline">
  <mat-label>名前</mat-label>
  <input matInput [formControl]="nameControl" />
  <mat-icon matSuffix>edit</mat-icon>
  <mat-error>必須項目</mat-error>
</mat-form-field>
```

`appearance` は `fill`（既定）と `outline` の2択。`subscriptSizing` で、エラー／ヒント行の高さを常に確保するか（`fixed`）、必要時のみ広げるか（`dynamic`）を切り替える。

中に置けるコントロールは `matInput`、`mat-select`、`mat-chip-grid`、`mat-date-range-input` など。

### 内部スロット

| 識別子 | 形 | 数 | 役割 |
| --- | --- | ---: | --- |
| `mat-label` | 要素 | 77 | ラベル。未入力時はプレースホルダ位置にあり、フォーカス／入力で枠線上へ浮き上がる |
| `mat-error` | 要素 | 64 | コントロールが invalid かつ touched のときだけ表示されるエラー文。複数置ける |
| `matHint` | 属性 | 1 | 下部の補助テキスト。`align="end"` で右寄せ |
| `matSuffix` | 属性 | 28 | 枠内の右端に任意の要素を置く |
| `matIconPrefix` | 属性 | 4 | 左端にアイコンを置く。`matPrefix` よりアイコン向けに余白が調整される |
| `matIconSuffix` | 属性 | 4 | 右端にアイコンを置く |

`mat-error` は自前で条件分岐せずとも表示タイミングを制御してくれる点が重要。逆に、`touched` になる前に出したい場合は `ErrorStateMatcher` を差し替える必要がある。

### `matInput`（属性・78 箇所）

`<input>` / `<textarea>` を `mat-form-field` のコントロールとして認識させるディレクティブ。`type`、`placeholder`、`required`、`disabled` などを Material 側の表示状態に橋渡しする。これを付け忘れると `mat-form-field` が実行時エラーを投げる。

---

## 5. セレクト・オートコンプリート

### `mat-select` / `mat-option` / `mat-select-trigger`

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `mat-select` | 19 | ドロップダウン本体。`mat-form-field` の中に置く |
| `mat-option` | 44 | 選択肢。`mat-select` と `mat-autocomplete` で共用 |
| `mat-select-trigger` | 2 | 閉じている状態の表示だけを差し替えるスロット |

```html
<mat-form-field>
  <mat-label>種別</mat-label>
  <mat-select [(value)]="kind" multiple>
    <mat-select-trigger>{{ kind.length }} 件選択中</mat-select-trigger>
    <mat-option value="a">A</mat-option>
  </mat-select>
</mat-form-field>
```

`multiple` を付けると選択肢にチェックボックスが出て、値は配列になる。既定の表示は選択値のカンマ連結なので、「N 件選択中」のような要約表示にしたいときに `mat-select-trigger` を使う。

### `mat-autocomplete` / `matAutocomplete`

`mat-autocomplete`（要素・8 箇所）が候補パネルを定義し、`matAutocomplete`（属性・8 箇所）で `<input>` と結び付ける。`mat-select` と違い、入力テキストは自由で、候補は絞り込み表示されるだけ。

```html
<input matInput [matAutocomplete]="auto" [formControl]="query" />
<mat-autocomplete #auto="matAutocomplete" [displayWith]="display">
  <mat-option *ngFor="let o of filtered()" [value]="o">{{ o.name }}</mat-option>
</mat-autocomplete>
```

候補の絞り込みは自前で行う。`displayWith` はオブジェクトを値にしたときの表示文字列を決める関数。

---

## 6. 選択コントロール

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `mat-checkbox` | 29 | チェックボックス。`indeterminate` で第3状態を持てる |
| `mat-radio-group` | 4 | ラジオのグループ。`name` の共有と値のバインドを担う |
| `mat-radio-button` | 12 | ラジオの各選択肢 |
| `mat-slide-toggle` | 16 | オン／オフのトグルスイッチ |

`mat-checkbox` と `mat-slide-toggle` は機能上どちらも真偽値だが、トグルは「即座に効く設定」、チェックボックスは「後でまとめて確定するフォーム項目」という使い分けが慣例。

### `mat-slider` / `matSliderThumb`

`mat-slider`（9 箇所）はトラックだけを描く器で、実際の値はその中に置く `<input matSliderThumb>`（5 箇所）が持つ。つまみは `<input type="range">` そのもので、キーボード操作とアクセシビリティはブラウザ実装に乗る。

```html
<mat-slider min="0" max="100" step="1" discrete>
  <input matSliderThumb [(ngModel)]="value" />
</mat-slider>
```

範囲指定の場合は `matSliderStartThumb` と `matSliderEndThumb` を2つ置く。

---

## 7. メニュー

| 識別子 | 形 | 数 | 役割 |
| --- | --- | ---: | --- |
| `mat-menu` | 要素 | 30 | メニューパネルの定義。宣言した場所には何も描画されない |
| `mat-menu-item` | 属性 | 82 | 各項目。`<button>` / `<a>` に付ける |
| `matMenuTriggerFor` | 属性 | 46 | クリックでメニューを開くトリガー |
| `matMenu` | exportAs | 40 | `#menu="matMenu"` でパネル参照を取る |
| `matMenuTrigger` | exportAs | 19 | トリガー側の参照。`openMenu()` / `closeMenu()` をコードから呼ぶ用 |
| `matMenuContent` | 属性 | 2 | 中身を `ng-template` 化し、開いたときに初めて生成する |
| `matMenuTriggerData` | 属性 | 2 | `matMenuContent` のテンプレートに渡すコンテキスト |

```html
<button mat-icon-button [matMenuTriggerFor]="menu"><mat-icon>more_vert</mat-icon></button>
<mat-menu #menu="matMenu">
  <button mat-menu-item (click)="rename()">名前変更</button>
  <button mat-menu-item [matMenuTriggerFor]="sub">詳細</button>
</mat-menu>
```

項目に `matMenuTriggerFor` を付けるとサブメニューになる。`matMenuContent` は、項目数が多い／中身の計算が重いメニューで初期描画コストを避けるための仕組み。

---

## 8. ダイアログ

ダイアログは `MatDialog` サービスの `open()` で開く（インポート 95 箇所）。テンプレート側の識別子は中身のレイアウト用。

| 識別子 | 形 | 数 | 役割 |
| --- | --- | ---: | --- |
| `mat-dialog-actions` | 要素 12 / 属性 6 | 18 | 下部のボタン行。`align="end"` で右寄せ |
| `mat-dialog-close` | 属性 | 15 | 付けた要素のクリックでダイアログを閉じる。値を渡すと `afterClosed()` の戻り値になる |
| `mat-dialog-title` | 属性 | 1 | タイトル行。スクロール時に固定され、`aria-labelledby` も設定される |

```html
<h2 mat-dialog-title>削除の確認</h2>
<mat-dialog-content>本当に削除する？</mat-dialog-content>
<mat-dialog-actions align="end">
  <button mat-button mat-dialog-close>キャンセル</button>
  <button mat-flat-button [mat-dialog-close]="true">削除</button>
</mat-dialog-actions>
```

本プロジェクトでは `mat-dialog-content` が使われておらず、スクロール領域は独自のラッパーコンポーネントや CSS で組まれている。そのため、本来 `mat-dialog-content` が自動で付ける「スクロール時のヘッダ／フッタ区切り線」は効かない。

---

## 9. タブ

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `mat-tab-group` | 8 | タブバーと本文の切り替えを内包する |
| `mat-tab` | 15 | 1枚のタブ。`label` と中身を持つ |
| `mat-tab-label` | 3 | ラベルに任意のマークアップを使うスロット |
| `mat-stretch-tabs` | 8 | ラベルを等幅で引き伸ばすか（`false` で内容幅） |
| `mat-align-tabs` | 7 | `start\|center\|end` でラベル行の寄せ位置 |

```html
<mat-tab-group [mat-stretch-tabs]="false" mat-align-tabs="start">
  <mat-tab label="概要">…</mat-tab>
  <mat-tab>
    <ng-template mat-tab-label><mat-icon>warning</mat-icon> 警告</ng-template>
    …
  </mat-tab>
</mat-tab-group>
```

### `mat-tab-nav-bar` / `mat-tab-link` / `mat-tab-nav-panel`

見た目はタブだが、中身の切り替えをルータに任せる構成。`mat-tab-nav-bar`（1 箇所）がバー、`mat-tab-link` がリンク、`mat-tab-nav-panel`（1 箇所）が対応する本文領域で、`[tabPanel]` で結び付ける。URL がタブ状態と連動する画面はこちらを使う。

---

## 10. 開閉パネル

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `mat-expansion-panel` | 14 | 開閉するパネル1枚 |
| `mat-expansion-panel-header` | 14 | 常に見えるヘッダ。クリックで開閉 |
| `mat-panel-title` | 4 | ヘッダ内のタイトル部 |
| `matExpansionPanelContent` | 4 | 中身を遅延生成する |

```html
<mat-expansion-panel [expanded]="open">
  <mat-expansion-panel-header>
    <mat-panel-title>詳細設定</mat-panel-title>
  </mat-expansion-panel-header>
  <ng-template matExpansionPanelContent>…</ng-template>
</mat-expansion-panel>
```

`mat-accordion` で囲むと「同時に1つだけ開く」制御ができるが、本プロジェクトでは使われておらず、各パネルが独立している。

`matExpansionPanelContent` は 4 箇所で使われている。重い中身を閉じている間は生成しないための指定。

---

## 11. サイドナビ（ドロワー）

同じ構造が2系統ある。

| 系統 | 識別子 | 数 |
| --- | --- | ---: |
| drawer | `mat-drawer-container` / `mat-drawer` / `mat-drawer-content` | 11 / 11 / 11 |
| sidenav | `mat-sidenav-container` / `mat-sidenav` / `mat-sidenav-content` | 1 / 1 / 1 |

`sidenav` は `drawer` の上位互換で、ビューポート全体に対する固定配置（`fixedInViewport`）など、アプリ全体のナビゲーション向けの機能が足されている。画面内の一部領域に出し入れするパネルには `drawer` を使う。本プロジェクトの使用比は 11 : 1 で、ほぼ画面内パネル用途。

```html
<mat-drawer-container>
  <mat-drawer #d mode="over" [opened]="filterOpen">…</mat-drawer>
  <mat-drawer-content>…</mat-drawer-content>
</mat-drawer-container>
```

`mode` は3択。`over`（本文に重ねる・背後を暗くする）、`push`（本文を押し出す）、`side`（本文と並べて幅を分け合う）。

---

## 12. リスト

| 識別子 | 形 | 数 | 役割 |
| --- | --- | ---: | --- |
| `mat-list` | 要素 | 1 | 表示専用のリスト |
| `mat-list-item` | 要素 4 / 属性 5 | 9 | 1行。`<a mat-list-item>` の形でリンクにもできる |
| `mat-nav-list` | 要素 | 2 | ナビゲーション用リスト。項目のホバー／フォーカス表現が付く |
| `mat-selection-list` | 要素 | 1 | 選択可能なリスト。フォームコントロールとして値を持つ |
| `mat-list-option` | 要素 | 1 | `mat-selection-list` の選択肢。チェックボックス付き |

`mat-selection-list` は、選択肢が多いときに `mat-select` の代わりに常時展開した状態で見せる用途に向く。

---

## 13. ツリー

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `mat-tree` | 2 | ツリー本体。データソースと対応付ける |
| `mat-tree-node` | 5 | 1ノードの行 |
| `matTreeNodeDef` | 5 | ノードのテンプレート定義。`when` 述語で種類ごとに出し分ける |
| `matTreeNodePadding` | 5 | 階層の深さに応じた左インデントを自動で付ける |
| `matTreeNodePaddingIndent` | 4 | 1階層あたりのインデント量（px） |
| `matTreeNodeToggle` | 1 | クリックでそのノードを開閉する |

```html
<mat-tree [dataSource]="source" [treeControl]="control">
  <mat-tree-node *matTreeNodeDef="let node" matTreeNodePadding [matTreeNodePaddingIndent]="16">
    <button mat-icon-button matTreeNodeToggle><mat-icon>chevron_right</mat-icon></button>
    {{ node.name }}
  </mat-tree-node>
</mat-tree>
```

ツリーは「平坦化したノード配列 + 深さ情報」で描く方式（flat tree）と、入れ子構造をそのまま描く方式（nested tree）がある。`matTreeNodePadding` は前者専用。

---

## 14. チップ

用途によって3系統に分かれており、本プロジェクトでは2系統が使われている。

### 選択用: `mat-chip-listbox` / `mat-chip-option`

`mat-chip-listbox`（3 箇所）の中に `mat-chip-option`（2 箇所）を並べ、タグを選択 UI として使う。`multiple` で複数選択。

### 入力用: `mat-chip-grid` / `mat-chip-row` / `matChipInputFor`

テキスト入力でタグを追加していく UI。

| 識別子 | 役割 |
| --- | --- |
| `mat-chip-grid` | `mat-form-field` のコントロールとして振る舞う器 |
| `mat-chip-row` | 追加済みの各チップ。編集・削除ができる |
| `matChipInputFor` | `<input>` をどの `mat-chip-grid` に紐づけるか |
| `matChipInputSeparatorKeyCodes` | 確定キー（Enter、カンマなど）のキーコード配列 |
| `matChipInputTokenEnd` | 確定時に発火する出力。ここでチップを配列に追加する |
| `matChipRemove` | 付けた要素のクリックでそのチップを削除する |

```html
<mat-form-field>
  <mat-chip-grid #grid>
    <mat-chip-row *ngFor="let t of tags" (removed)="remove(t)">
      {{ t }}
      <button matChipRemove><mat-icon>cancel</mat-icon></button>
    </mat-chip-row>
  </mat-chip-grid>
  <input [matChipInputFor]="grid"
         [matChipInputSeparatorKeyCodes]="separators"
         (matChipInputTokenEnd)="add($event)" />
</mat-form-field>
```

表示専用の `mat-chip-set` / `mat-chip` は本プロジェクトでは使われていない。

---

## 15. 日付選択

| 識別子 | 形 | 数 | 役割 |
| --- | --- | ---: | --- |
| `mat-datepicker` | 要素 | 2 | カレンダーパネルの定義。宣言位置には描画されない |
| `mat-datepicker-toggle` | 要素 | 3 | カレンダーを開くボタン。`[for]` でパネルを指定 |
| `matDatepicker` | 属性 | 2 | `<input>` とパネルを結ぶ |
| `mat-date-range-input` | 要素 | 1 | 開始／終了の2入力をまとめたコントロール |
| `matStartDate` / `matEndDate` | 属性 | 1 / 1 | 範囲入力の開始側・終了側の `<input>` |
| `mat-date-range-picker` | 要素 | 1 | 範囲選択用のカレンダーパネル |

```html
<mat-form-field>
  <mat-label>期間</mat-label>
  <mat-date-range-input [rangePicker]="picker">
    <input matStartDate [formControl]="from" />
    <input matEndDate [formControl]="to" />
  </mat-date-range-input>
  <mat-datepicker-toggle matIconSuffix [for]="picker"></mat-datepicker-toggle>
  <mat-date-range-picker #picker></mat-date-range-picker>
</mat-form-field>
```

日付の型（`Date` か Luxon か等）は `provideNativeDateAdapter()` などの DateAdapter 設定で決まる。これを入れないと実行時エラーになる。

---

## 16. ステッパー

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `mat-stepper` | 1 | 多段の入力フローを管理 |
| `mat-step` | 5 | 1ステップ。`stepControl` に `FormGroup` を渡すと妥当性で進行を制御できる |
| `matStepperNext` | 4 | 次ステップへ進めるボタン |
| `matStepperPrevious` | 4 | 前ステップへ戻るボタン |
| `matStepperIcon` | 1 | ステップ番号の丸を任意のアイコンに差し替える |

`mat-stepper` は `orientation="horizontal\|vertical"` で向きを切り替える（旧来の `mat-horizontal-stepper` / `mat-vertical-stepper` の統合形）。`linear` を付けると、現在のステップが valid になるまで先へ進めない。

---

## 17. 進捗表示

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `mat-spinner` | 18 | 進捗不定の円形スピナー |
| `mat-progress-spinner` | 2 | 円形スピナー本体。`mode="determinate"` で `value` による進捗表示ができる |
| `mat-progress-bar` | 1 | 横棒の進捗バー |

`mat-spinner` は `mat-progress-spinner` に `mode="indeterminate"` を固定したのと等価な別名セレクタ。実装は同一コンポーネント。使用比が 18 : 2 であることから、ほぼ「読み込み中」表示専用に使われている。

`mat-progress-bar` の `mode` は `determinate`（進捗）、`indeterminate`（不定）、`buffer`（二段表示）、`query`（待機→進捗の前段）の4つ。

---

## 18. 区切り線

### `mat-divider`（8 箇所）

1px の罫線。`vertical` で縦線、`inset` で左に余白を空けた線になる。`mat-list` や `mat-menu` の中に置くと、その文脈に合った余白が自動で付く。

---

## 19. ツールチップ

`matTooltip` そのものはテンプレートに現れないが、`MatTooltip` を継承した独自ディレクティブ経由で全面的に使われている。そのため MatTooltip の入力はそのまま有効。

| 識別子 | 数 | 役割 |
| --- | ---: | --- |
| `matTooltipShowDelay` | 38 | 表示までの遅延（ms） |
| `matTooltipPosition` | 26 | `above\|below\|left\|right\|before\|after` |
| `matTooltipDisabled` | 3 | 一時的に無効化する |
| `matTooltipClass` | 1 | パネルに付与する CSS クラス |

ラッパー側では、スクロール時の挙動（`MAT_TOOLTIP_SCROLL_STRATEGY`）を差し替え、表示種別に応じたクラスを自動で付けている。ツールチップに独自の見た目を一括適用したい場合、この種のラッパーを1つ作って全箇所で使うのが定石。

---

## 20. 属性形ディレクティブ・`exportAs` 一覧

要素タグ以外の識別子をまとめて再掲する。

### 属性セレクタ

| 識別子 | 数 | 対象モジュール |
| --- | ---: | --- |
| `mat-icon-button` | 118 | button |
| `mat-menu-item` | 82 | menu |
| `matInput` | 78 | input |
| `mat-flat-button` | 54 | button |
| `matMenuTriggerFor` | 46 | menu |
| `mat-stroked-button` | 40 | button |
| `matTooltipShowDelay` | 38 | tooltip |
| `mat-button` | 29 | button |
| `matSuffix` | 28 | form-field |
| `matTooltipPosition` | 26 | tooltip |
| `mat-dialog-close` | 15 | dialog |
| `matAutocomplete` | 8 | autocomplete |
| `mat-stretch-tabs` | 8 | tabs |
| `mat-align-tabs` | 7 | tabs |
| `mat-dialog-actions` | 6 | dialog |
| `matTreeNodeDef` | 5 | tree |
| `matTreeNodePadding` | 5 | tree |
| `matSliderThumb` | 5 | slider |
| `mat-list-item` | 5 | list |
| `matTreeNodePaddingIndent` | 4 | tree |
| `matStepperNext` | 4 | stepper |
| `matStepperPrevious` | 4 | stepper |
| `matIconPrefix` | 4 | form-field |
| `matIconSuffix` | 4 | form-field |
| `matExpansionPanelContent` | 4 | expansion |
| `matTooltipDisabled` | 3 | tooltip |
| `mat-tab-label` | 3 | tabs |
| `matMenuContent` | 2 | menu |
| `matMenuTriggerData` | 2 | menu |
| `matDatepicker` | 2 | datepicker |
| `matTreeNodeToggle` | 1 | tree |
| `matTooltipClass` | 1 | tooltip |
| `matStepperIcon` | 1 | stepper |
| `matStartDate` / `matEndDate` | 1 / 1 | datepicker |
| `matHint` | 1 | form-field |
| `matChipRemove` | 1 | chips |
| `matChipInputFor` | 1 | chips |
| `matChipInputSeparatorKeyCodes` | 1 | chips |
| `matChipInputTokenEnd` | 1 | chips（出力） |
| `matBadge` / `matBadgePosition` / `matBadgeSize` | 1 / 1 / 1 | badge |
| `mat-tab-nav-bar` | 1 | tabs |
| `mat-dialog-title` | 1 | dialog |

### `exportAs` 名

`#ref="..."` の形でインスタンスを取り出すための名前。

| 識別子 | 数 | 取り出せるもの |
| --- | ---: | --- |
| `matMenu` | 40 | `MatMenu`。`matMenuTriggerFor` に渡す |
| `matMenuTrigger` | 19 | `MatMenuTrigger`。`openMenu()` / `closeMenu()` |
| `matAutocomplete` | 8 | `MatAutocomplete`。`[matAutocomplete]` に渡す |

---

## 21. CSS クラスとしての `mat-*`

タグでもディレクティブでもないが、テンプレート内の `class` 属性に現れるもの。

| クラス | 数 | 内容 |
| --- | ---: | --- |
| `mat-elevation-z0` | 1 | Material が提供する影のユーティリティクラス。`z0`〜`z24` で影の強さを指定。`z0` は影を消す目的 |
| `mat-light` | 2 | Material 製ではなく、本プロジェクト独自のテーマ用クラス |

---

## 22. 使われていない主要モジュール

参考として、Angular Material に存在するが本プロジェクトでインポートされていないもの。

`table`（`mat-table`）、`card`、`toolbar`、`snack-bar`、`paginator`、`sort`、`grid-list`、`bottom-sheet`、`timepicker`、`ripple`。

表組みに `mat-table` を使わず独自のテーブル実装を持っている点、通知に `snack-bar` を使っていない点が構成上の特徴。同種の UI を追加するときは、Material を持ち込むより既存の独自実装に合わせたほうが見た目の整合性を保てる。

---

## 付録: 抽出する場合に使えるコマンド

```bash
# 要素タグ（開始タグ）の種類と出現数
grep -rhoE '<mat-[a-zA-Z0-9-]+' --include='*.html' --include='*.ts' src/ \
  | sed 's/<//' | sort | uniq -c | sort -rn

# 属性形ディレクティブの種類と出現数
grep -rhoP '(^|\s)\[?(mat[A-Z][a-zA-Z0-9]*|mat-[a-z-]+)\]?(?==|\s|>)' --include='*.html' src/ \
  | tr -d ' []' | sort | uniq -c | sort -rn

# インポートされている Material モジュール
grep -rhoE "from '@angular/material/[a-z-]+'" --include='*.ts' src/ \
  | sort | uniq -c | sort -rn
```

単純な文字列一致なので、`timeFormatString` の `matString`、`formatY()` の `matY` のような誤検出が混ざる。本資料では個別に確認して除外済み。
