# MemoryCard

  Angular・TypeScript・RxJSで、忘れやすい違いを思い出すためのメモ。
  「誰が持つか」「何がきっかけか」「どこへ伝わるか」を軸に読む。
  API例はAngular 22系・NgRx 22系・RxJS 7.8系を想定。
  コードは要点の抜粋。importや周囲の定義は省略している。

## 1. コードリーディング

### 最初に「何を確かめたいか」を決める

  「保存を押したあと、一覧が更新されるまで」のように、操作と終点を決める。
  画面全体を順に読むより、1つの操作の経路を追う。

  ```text
  画面の操作 → テンプレートのイベント → メソッド
             → 状態更新／API通信 → 表示に使う値 → 画面
  ```

  - メソッドに着いたら、入力・状態変更・戻り値を見る。
  - 正常時だけでなく、失敗・キャンセル・画面離脱の経路も見る。
  - IDEの「定義へ移動」で処理先、「参照を検索」で呼び出し元を探す。
  - 変数のつながりは参照検索やハイライトで確認する。

### 画面の部品とコードを結びつける

  1. ブラウザの開発者ツールで、対象のボタンや入力欄を選ぶ。
  2. 近くの`app-*`タグ、`data-*`属性、表示文字列を確認する。
  3. コード内で検索し、テンプレートの`(click)`などからメソッドへ進む。

  Angular DevToolsが使える環境なら、画面からコンポーネントを選んで辿れる。
  ブラウザの要素ツリーはDOM、Angular DevToolsのツリーはコンポーネントの関係を示す。

  - `<app-editor>`を見つけたら、`selector: 'app-editor'`を検索する。
  - 属性の値が見つからなければ、`[attr.data-id]="item.id"`のような動的生成を疑う。
  - 表示文字列が見つからなければ、翻訳キーやAPIから来た値も確認する。
  - コピーしたCSSセレクターは画面上の位置。コード内に同じ文字列があるとは限らない。

### 子から親へ処理を追う

  子で`output.emit(...)`を見つけたら、子のセレクターを使う親のテンプレートを探す。

  ```html
  <app-editor (saved)="refresh($event)" />
  ```

  子の`saved.emit(...)` → 親の`refresh(...)`へ続く。
  親から子への値は`[value]="..."`、双方向の値は`[(value)]="..."`を見る。

### 親のHTMLに子のタグがないとき

  ```text
  静的な配置      → 親のテンプレートに <app-detail>
  ルートで配置    → <router-outlet> とルート設定
  差し込む中身    → <ng-content> と呼び出し側のHTML
  動的な配置      → ngComponentOutlet／createComponent／ダイアログ
  ```

  `router-outlet`なら、`children`・`component`・`loadComponent`を確認する。
  DOMの入れ子だけでは、Componentをどう配置したかは分からない。
  また、importの依存図だけでは、画面上の親子関係は確定できない。

### 状態管理に出会ったら、次に何を見るか

| 見つけたもの | 次に見るもの |
| --- | --- |
| `store.dispatch(A(...))` | Aの定義、全ての`on(A)`・`ofType(A)` |
| `createEffect()` | 起点のAction、呼ぶService、結果のAction |
| `createReducer()` | State型、初期値、どのActionで何を変えるか |
| `store.select(S)`／`selectSignal(S)` | Sの定義、元のState、更新元 |
| `createSelector(...)` | 入力Selector、計算内容、利用側 |
| Signalの`.set()`／`.update()` | 呼び出し元、そのSignalを読む箇所 |
| `computed()`／`effect()` | 中で読むSignal、結果や副作用の利用先 |
| SignalStoreのメソッド | 呼び出し元、`patchState()`やService呼び出し |
| SignalStoreのEvent | 発火元、全ての`on()`・`events.on()`、届く範囲 |
| 独自の`withXxx()`やFacade | 中の実装。名前だけで判断しない |

  **ActionやEventの受け手は1か所とは限らない。** 状態変更と通信が別々に反応することもある。
  Providerの配置も確認する。同じクラス名でも、別のインスタンスを使っている場合がある。

### 型を見るタイミング

  データの意味や形が変わるところで型定義を見る。
  Stateを追うならState型と初期値、APIを追うなら要求・応答の型から確認する。

  ```text
  APIの形式（DTO） → 変換処理 → アプリ内のデータ → 表示用の値
  ```

  名前が同じでも、途中でキー名・単位・欠損値の扱いが変わることがある。
  Angularの`model()`と、業務データを表すModel型は別の意味。

### URL・通信・ダイアログを追う

  - URL：`params`はパス中の値、`queryParams`は`?`以降の値。
  - 一覧条件：操作前後のURLを比べ、フィルタや並び順を読む処理・書く処理を探す。
  - 通信：NetworkでAPIのパス・要求・応答を確認し、Serviceの通信処理と照合する。
  - ダイアログ：`dialog.open(DialogComponent)`で開く箇所を探す。
  - 閉じる処理：中の`MatDialogRef.close(result)`から、呼び出し元の`afterClosed()`へ進む。

### 迷ったときの確認

  「誰が起こした？ → 何を変えた？ → 誰が読んだ？ → いつ終わる？」
  この4点を説明できれば、その操作の大筋は掴めている。

## 2. コンポーネントとテンプレート

### コンポーネントを分ける基準

  **再利用回数より、責務が独立しているか。** 1画面だけの部品でも分けてよい。
  例：検索条件・一覧・詳細パネルは、それぞれ変更理由が違う。

  - Componentに残す：表示・選択・開閉・入力受付など、画面の都合。
  - 外へ出す：業務ルール・データ変換・通信など、画面がなくても必要な処理。
  - 共通化する：似た見た目だけでなく、同じ責務と振る舞いを持つとき。

### input・output・model：宣言するのは子

| API | 流れ | 子の役割 |
| --- | --- | --- |
| `input()` | 親 → 子 | 渡された値を読む |
| `output()` | 子 → 親 | `.emit()`で出来事を知らせる |
| `model()` | 親 ↔ 子 | 値を読み、自分からも変更する |

  親のテンプレートでは、**左が子の公開名、右が親の値や処理**。

  ```html
  <app-counter
    [label]="title()"
    (saved)="save($event)"
    [(count)]="count"
  />
  ```

  - 子の宣言例：`label = input('')`、`saved = output<number>()`、`count = model(0)`。
  - 親の`title()`はSignalの現在値。`$event`は子が`.emit()`した値。
  - 親の`count`が書き込み可能なSignalなら、`[(count)]`には`count()`でなく`count`を渡す。
  - `model()`は対応する`countChange`も自動で用意する。普通の親プロパティとも結べる。

### viewChildとcontentChild：誰が書いた中身か

| API | 探す場所 | 例 |
| --- | --- | --- |
| `viewChild()` | 自分のテンプレート | 自分が配置した入力欄や子Component |
| `contentChild()` | 呼び出し側が渡した中身 | `<ng-content>`で受ける見出しやテンプレート |

  どちらも取得結果をSignalで読む。子Componentの内部まで自由に検索できるわけではない。

  ```ts
  readonly editor = viewChild(EditorComponent);

  focusEditor(): void {
    this.editor()?.focus();
  }
  ```

  - 主な用途：フォーカス・スクロール・子の公開メソッド呼び出し。
  - 値の受け渡しは`input()`・`output()`・`model()`を基本にする。
  - `@if`などで不在になり得る子は通常のqueryで扱う。不在なら`undefined`。
  - `.required()`は必ず存在する前提。未取得のタイミングで読むとエラーになる。
  - queryのSignalは読み取り専用。取得した子のメソッドは呼べるが、query自体を`.set()`できない。

### ng-content：呼び出し側の中身を、枠へ入れる

  親が中身を用意し、子が表示場所を用意する。

  ```html
  <!-- 親 -->
  <app-card>
    <h2 card-header>見出し</h2>
    <p>本文</p>
  </app-card>

  <!-- 子のテンプレート -->
  <ng-content select="[card-header]" />
  <ng-content />
  ```

  `[card-header]`は属性セレクター。見出しだけを先に振り分け、残りを通常の枠へ入れる。
  `select="app-chart"`なら、該当するタグを選ぶ。

### ng-container・ng-template：囲むものと、展開するもの

  - `ng-container`：DOM要素を増やさず、中身をまとめる。
  - `ng-template`：あとで展開するHTMLのひな形。その場所ではまだ表示されない。
  - `ngTemplateOutlet`：ひな形を指定した場所に展開するDirective。

  ```html
  <ng-template #row let-item let-index="index">
    <p>{{ index }}：{{ item.title }}</p>
  </ng-template>

  <ng-container
    [ngTemplateOutlet]="row"
    [ngTemplateOutletContext]="{ $implicit: item, index: 0 }"
  />
  ```

  `NgTemplateOutlet`をComponentの`imports`に加えて使う。
  `context`はひな形へ渡すデータ。`$implicit`は`let-item`、`index`は`let-index="index"`に入る。

### ng-templateを子へ渡す理由

  **繰り返しや表示タイミングは子、中身の見た目は親が決めるため。**
  子は`contentChild(TemplateRef)`などで受け取り、データを変えて複数回展開できる。

  - 一覧の枠・ページングは共通にし、1行の描画だけ変える。
  - ダイアログの枠は共通にし、本文だけ変える。
  - Loading・Emptyの表示を呼び出し側で変える。

  `ng-content`は用意した中身の配置、`ng-template`は展開方法も受け手が制御する。

### 属性バインディング：プロパティとHTML属性

  ```html
  <button [disabled]="busy()" [attr.aria-busy]="busy()">保存</button>
  <p [attr.data-status]="status()">状態</p>
  ```

  `[disabled]`はDOMプロパティ、`[attr.data-status]`はHTML属性を設定する。
  `attr.`は`aria-*`・`data-*`・`colspan`などに使う。値が`null`なら属性を削除する。

  `data-status="warning"`の要素は、CSSや`querySelector('[data-status="warning"]')`で探せる。
  独自データには`data-*`、投影の目印には`card-header`のような属性も使える。

### Directive：要素に振る舞いを付ける

  `@Component()`はテンプレートを持つ画面部品。
  `@Directive()`は既存の要素へ機能を付ける。例：強調表示・入力補助・クリック制御。
  `<div appHighlight>`なら、`selector: '[appHighlight]'`のDirectiveを探す。

### @defer：必要になったら読み込む

  初期表示を軽くするため、条件や操作などをきっかけに部品を読み込み・描画する。
  何も指定しない場合は、ブラウザがアイドルになったときに開始する。

  - `@placeholder`：開始前の表示。
  - `@loading`：部品を読み込んでいる間の表示。
  - `@error`：部品の読み込みに失敗したときの表示。

  `when`が一度成立すると、後でfalseになっても元には戻らない。
  表示の切り替えには`@if`を使う。遅延ロードにはstandaloneなどの条件もある。

## 3. Signalと非同期データ

### SignalとObservable：現在値か、時間の流れか

  - Signal：**今の値**を`count()`のように読む。状態や表示の計算に向く。
  - Observable：**時間とともに届く値**を扱う。通信・イベント・待機・キャンセルに向く。

  判断は「変換できるか」より、「変換すると読み方が単純になるか」。
  RxJSの加工を続けるならObservable、画面で現在値を読むならSignalが扱いやすい。

### signal・computed・linkedSignalの違い

| API | 自分で変更 | 他のSignalから計算 | 覚え方 |
| --- | --- | --- | --- |
| `signal()` | ○ | 自動ではしない | 自分が持つ状態 |
| `computed()` | × | ○ | 状態から求める計算結果 |
| `linkedSignal()` | ○ | ○ | 元の状態に連動し、手動でも変えられる値 |

  ```ts
  const count = signal(0);
  const doubled = computed(() => count() * 2);

  count.set(3);
  count.update(value => value + 1);
  doubled(); // 8
  ```

  `computed()`は中で読んだSignalを追跡し、必要なときに再計算する。
  配列やオブジェクトは直接書き換えず、新しい値を`.set()`／`.update()`で渡す。

### readonlyとasReadonly：禁止するものが違う

  ```ts
  private readonly countState = signal(0);
  readonly count = this.countState.asReadonly();
  ```

  - `readonly`：`this.count = 別のSignal`というプロパティの再代入を禁止する。
  - `asReadonly()`：公開側から`.set()`・`.update()`を使えなくする。
  - 内部の`countState`は更新でき、その結果を公開側も読む。

  どちらもオブジェクト内部を凍結しない。入れ子の値の直接変更まで防ぐものではない。

### linkedSignal：選択を保ち、必要なら選び直す

  「一覧は更新されるが、選択中の項目は残っている限り維持したい」ときに使う。

  ```ts
  type Item = { id: number };
  const items = signal<Item[]>([{ id: 1 }, { id: 2 }]);
  const selected = linkedSignal<Item[], Item | undefined>({
    source: items,
    computation: (current, previous) =>
      current.find(item => item.id === previous?.value?.id) ?? current[0],
  });
  ```

  - `source`：連動元。変わると選択値を再計算する。
  - `previous?.value`：前回の選択値。手動で変更した値も含む。
  - `previous?.source`：前回の連動元。
  - `selected.set(...)`で選択変更もできる。次の連動元の変更時には再計算される。

  元の値が変わるたびに`computed()`内で新しいSignalを作ると、状態が作り直される。
  選択の維持・リセットなら、まず`linkedSignal()`で表せるか考える。

### effect：値を求めるのでなく、何かを実行する

  **中で読んだSignalが変わったら、外部への処理を再実行する。** 初回も実行される。
  例：ログ出力・保存先との同期。NgRxのEffectsとは別の仕組み。

  ```ts
  constructor() {
    effect((onCleanup) => {
      const value = this.count();
      const timerId = setTimeout(() => console.log(value), 1000);
      onCleanup(() => clearTimeout(timerId));
    });
  }
  ```

  - `onCleanup`：再実行前やeffect破棄時に、前回のタイマー・購読などを片付ける。
  - `untracked(otherSignal)`：値は読むが、その読み取りを依存として追跡しない。
  - `untracked(() => 処理)`：処理中のSignal読み取りをまとめて追跡対象から外す。

  派生値なら`computed()`、連動しつつ変更可能なら`linkedSignal()`。
  Signalから別Signalへ値をコピーするだけのeffectは、状態の持ち方を見直す。
  DOM更新後の計測・操作には`afterRenderEffect()`を検討する。

### toSignal：Observableの最新値をSignalで読む

  ```ts
  readonly keyword = toSignal(this.control.valueChanges, {
    initialValue: this.control.value,
  });
  ```

  `keyword()`で読む。Resourceのような`keyword.value()`ではない。
  作成時に購読し、通常は作成したComponent／Serviceの破棄に合わせて解除する。

  - 初期値なしなら、最初の通知まで`undefined`になり得る。
  - `requireSync: true`は、購読直後に同期的な値が必ず来る場合に使う。
  - 元のObservableがエラーになると、Signalを読んだときにエラーが投げられる。
  - 同じObservableに対して何度も作らず、一度作ったSignalを使い回す。

  逆方向の`toObservable()`は、Signalの変化をRxJSで加工したいときに使う。
  `effect()`・`toSignal()`などは、通常Component／Serviceのフィールド初期化やconstructorで作る。

### Resource：取得結果・待機・失敗をまとめて持つ

| API | 取得処理 | 頭の中での意味 |
| --- | --- | --- |
| `resource()` | Promiseを返す`loader` | 非同期処理の状態管理 |
| `rxResource()` | Observableを返す`stream` | RxJSによる取得の状態管理 |
| `httpResource()` | URLや要求設定 | HTTP取得の状態管理 |
| `toSignal()` | 既存のObservable | 最新値の読み方を変える |

  Resourceは取得条件の変化に追従し、結果・loading・errorをSignalとして公開する。
  基本はデータ取得用。保存・削除などの更新操作には使わない。

  ```ts
  readonly itemId = signal(1);
  readonly item = rxResource({
    params: () => this.itemId(),
    stream: ({ params }) => this.api.getItem(params),
  });
  ```

  `this.api.getItem()`はObservableを返すServiceのメソッド。
  `stream`では必ずObservableを返す。`{ ... }`を書くなら`return`が必要。
  値を出さずに完了するとエラーになるため、ここで`EMPTY`に差し替えない。

  - `item.value()`：取得した値。失敗状態ではエラーを投げるため、`hasValue()`で確認して読む。
  - `item.isLoading()`：取得中か。
  - `item.error()`：エラー内容。
  - `item.status()`：現在の状態。
  - `item.reload()`：条件を変えずに再取得する。

  `params`が`undefined`なら取得を行わない。
  条件変更時は前の取得をキャンセルする。Promise側の処理には`abortSignal`を渡すなどの対応が必要。

### httpResource：HTTPをSignalの状態として扱う

  ```ts
  readonly item = httpResource<{ id: number; title: string }>(
    () => `/api/items/${this.itemId()}`,
  );
  ```

  `@angular/common/http`から使う。`provideHttpClient()`の設定が必要。
  HttpClientの通信経路を使うため、Interceptorも適用される。
  型引数は実データの検証ではない。応答の内容チェックは別途必要。

## 4. 状態管理

### どこへ状態を置くか

| 共有したい範囲・目的 | 最初に考えるもの |
| --- | --- |
| 1つのComponent内 | `signal()` |
| 親子や少数の兄弟 | 共通の親へ状態を持ち上げ、input／outputで接続 |
| 複数Componentで共有 | Service＋Signal、またはSignalStore |
| Actionを軸に変更や副作用を管理 | NgRx Store＋Effects |

  Storeを使うかは規模だけでなく、変更理由の記録・共有範囲・既存設計から決める。
  ServiceやSignalStoreの共有範囲と寿命は、Providerを置く場所で変わる。

### NgRx Store：出来事から状態を変える

  ```text
  操作 → dispatch(Action) → Reducer → State → Selector → 画面
                         → Effect → API → 結果Action → Reducer
  ```

  - Action：何が起きたかと、必要なデータ。
  - Reducer：Actionに応じて新しいStateを返す。通信などの副作用は行わない。
  - Effect：Actionを受け、通信・URL変更などを行う。
  - Selector：Stateから表示に必要な値を取り出す・計算する。

  上の図は関係の整理。ActionはReducerで処理された後、Effectsへ流れる。
  `createEffect()`の購読はNgRxが管理するため、通常その中に`subscribe()`は書かない。

### createSelectorは「計算を定義」、selectは「結果を読む」

  ```ts
  const selectDoneCount = createSelector(
    selectItems,
    items => items.filter(item => item.done).length,
  );
  ```

  `createSelector()`は入力Selectorから値を求める関数を作る。
  入力が同じなら前回の計算結果を再利用する。Signalの`computed()`に近い役割。

  - `store.select(selectDoneCount)`：結果をObservableで受け取る。
  - `store.selectSignal(selectDoneCount)`：結果をSignalで読む。
  - `select().pipe(...)`：その後もRxJSで加工したいとき。
  - `select(...)`＋`async`：テンプレートで購読・解除を任せたいとき。

### createFeatureSelector：Stateの「どの区画か」を決める

  ```ts
  const selectListState = createFeatureSelector<ListState>('list');
  const selectItems = createSelector(selectListState, state => state.items);
  ```

  全体Stateから`list`区画を取る、共通の入口を作っている。
  `(state: AppState) => state.list`でも表せるが、入口を共有すると区画名の重複を減らせる。
  `'list'`はStoreへの登録キーと一致させる。

### dispatch: false：結果をActionとして送らない

  `createEffect(..., { dispatch: false })`は、副作用だけで終えるEffectの指定。
  例：URL変更・通知表示・ログ。返した値をNgRxがActionとしてdispatchしなくなる。
  内部で明示的に呼ぶ`store.dispatch()`まで禁止する指定ではない。

### SignalStore：状態・計算・操作をまとめる

  `signalStore(...)`に必要な機能を組み合わせる。Action方式は必須ではない。

| API | 役割 |
| --- | --- |
| `withState()` | 状態と初期値を用意する |
| `withComputed()` | 状態から値を計算する |
| `withMethods()` | 状態を操作するメソッドを用意する |
| `patchState()` | 状態の一部を更新する |
| `rxMethod()` | 呼び出しをRxJSの処理につなぐ |
| `withLinkedState()` | 他の状態に連動し、変更もできる状態を用意する |
| `signalStoreFeature()` | 複数の機能を再利用できる1つの部品にまとめる |

  `signalStoreFeature()`は処理を別方式に変えるものではなく、設定をまとめるためのもの。
  1つのStoreでしか使わない小さな処理は、直接並べてもよい。

### SignalStore Events：メソッド以外に「出来事」でもつなげる

| NgRx Store | SignalStore Eventsの対応する役割 |
| --- | --- |
| `createActionGroup()`／`props<T>()` | `eventGroup()`／`type<T>()` |
| `store.dispatch()` | `injectDispatch()`／`Dispatcher.dispatch()` |
| `createReducer()`＋`on()` | `withReducer()`＋`on()` |
| `Actions`＋`ofType()` | `Events`＋`events.on()` |
| Effectで副作用を処理 | `withEventHandlers()`などで副作用を処理 |

  Eventの定義だけでは処理は動かない。発火する側と受ける側を接続する。
  `provideDispatcher()`・`toScope()`がある場合は、Eventがどの範囲に届くかも確認する。

## 5. RxJS

### 作る・加工する・受け取る

  ```text
  of／fromなど → Observableを作る
  pipe(...)   → 流れる値やタイミングを加工する
  subscribe() → 通知を受け取る
  ```

  Observableには、値の通知`next`・正常終了`complete`・失敗`error`がある。
  `unsubscribe()`は購読解除。`complete`の通知を受けたこととは違う。
  HttpClientなど、購読を始めると処理が動くObservableもある。

### Observableを作る関数

| API | 何が流れるか |
| --- | --- |
| `of([10, 20])` | 配列`[10, 20]`が1回 |
| `from([10, 20])` | `10`・`20`が順に2回 |
| `from(promise)` | Promiseの解決値。失敗ならエラー |
| `fromEvent(element, 'click')` | クリックのたびにイベント |
| `interval(1000)` | 1秒後から1秒ごとに`0, 1, 2...` |
| `timer(1000)` | 1秒後に`0`を1回流して完了 |
| `timer(0, 5000)` | ほぼ即時に`0`、以降5秒ごとに`1, 2...` |
| `throwError(() => error)` | 値を流さずエラー通知 |
| `EMPTY` | 値を流さず完了 |
| `iif(condition, a$, b$)` | 購読開始時の条件でa$かb$を選ぶ |

  `iif()`は条件を監視し続けるものではない。
  `from(promise)`も、すでに動いているPromiseの処理を購読解除で止めるわけではない。

### SubjectとBehaviorSubject

  - `Subject`：`.next()`した値を、その時点の購読者へ流す。現在値は保持しない。
  - `BehaviorSubject`：初期値が必要。新しい購読者にも、保持している最新値を渡す。

  `combineLatest()`は全ての入力から最低1回値が必要なので、初期値の有無が効く。

### 既存のObservableをどう合流するか

| API | 結果 | 待つ条件 |
| --- | --- | --- |
| `merge(a$, b$)` | 各値が来た順に1本へ流れる | 両方を同時に購読 |
| `concat(a$, b$)` | a$の全値、そのあとb$の全値 | a$の完了後にb$を購読 |
| `combineLatest([a$, b$])` | `[最新A, 最新B]`を繰り返し流す | 全員が1回流した後、誰かの更新で通知 |
| `a$.pipe(combineLatestWith(b$))` | 上と同じ最新値の組 | pipeで書く形 |
| `forkJoin([a$, b$])` | `[最後のA, 最後のB]`を1回 | 全員が値を出して完了 |

  `forkJoin()`は複数HTTPの結果を揃えるときに便利。
  1つでも終わらないと結果が揃わない。RxJS 7.8では、空のまま完了する入力があると結果なしで完了する。

### 流れてきた値から、新しい処理を作る

  `merge()`などは既存の流れを合流する。`〇〇Map()`は各値からObservableを作って処理する。

| API | 新しい入力が来たら | 主な用途 |
| --- | --- | --- |
| `switchMap()` | 前の購読を解除し、最新の処理へ切り替える | 検索条件の変更 |
| `concatMap()` | 前の処理が終わるまで待つ | 保存を順番に行う |
| `mergeMap()` | 並行して処理する。結果の順番は保証しない | 独立した処理の同時実行 |
| `exhaustMap()` | 実行中の新しい入力を無視する | 送信ボタンの連打防止 |

  `switchMap()`の解除で止まるのは購読。サーバで始まった保存処理まで取り消せるとは限らない。

### withLatestFromとconcatLatestFrom

  **どちらも、主役の値が来たときだけ、別の最新値を添える。**
  別の値だけが更新されても通知しない。ここが`combineLatest()`との違い。

  - `withLatestFrom()`：RxJS。別のObservableを先に購読しておく。
  - `concatLatestFrom()`：`@ngrx/operators`。主役が来たときに購読先を作る。
  - NgRx EffectでActionの情報からSelectorを選ぶなら、`concatLatestFrom(action => ...)`が便利。

  添える側にまだ値がないと、その主役の通知を取りこぼすことがある。
  `concatLatestFrom()`へ、応答待ちのHTTPをそのまま渡すのは避ける。

### 時間で間引く：止まるまで待つか、一定間隔で取るか

| API | 出すタイミング | 500ms内にA→B→Cなら |
| --- | --- | --- |
| `debounceTime(500)` | 最後の入力から500ms止まった後 | C |
| `throttleTime(500)` | 最初をすぐ出し、その後500ms抑える | A（標準設定） |
| `auditTime(500)` | 最初の入力から500ms後に、その間の最後を出す | C |
| `debounce(value => timer(...))` | 値ごとに待ち時間を変える | 指定した待ち時間による |

  検索文字の入力待ちは`debounceTime()`、連続する座標の定期取得は`auditTime()`。
  `throttleTime()`は時間で抑えるため、処理中ずっと連打を防ぐ用途なら`exhaustMap()`も検討する。

### 値の加工・比較

| Operator | 覚え方・例 |
| --- | --- |
| `map(fn)` | 各値を変換する |
| `filter(fn)` | 条件に合う値だけ流す |
| `tap(fn)` | 値を変えずにログなどを実行する |
| `distinctUntilChanged()` | 直前と同値なら流さない。標準は`===` |
| `distinctUntilKeyChanged('id')` | 直前と同じidなら流さない |
| `startWith(x)` | 元の値より先にxを流す |
| `skip(n)` | 最初のn件を無視する |
| `pairwise()` | 前回と今回を組にする。最初の1件だけでは流れない |
| `delay(ms)` | 値の通知を遅らせる |
| `timeout(ms)` | 指定時間内に次の値が来なければエラー |
| `bufferTime(ms)` | 一定時間の値を配列にまとめる |

  `distinctUntilChanged()`は全履歴の重複排除ではない。`A → B → A`は3件とも流れる。
  オブジェクトは同じ内容でも別の参照なら流れるため、必要なら比較関数を渡す。
  `skip(1)`は「ユーザー操作だけ」の保証ではなく、単に最初の通知を捨てる。

### take(1)とfirst：空で終わったときが違う

| API | 取得 | 1件も取得せず完了したら |
| --- | --- | --- |
| `take(1)` | 最初の1件 | そのまま完了 |
| `first()` | 最初の1件。条件も指定できる | デフォルト値がなければ`EmptyError` |

  `first(predicate, defaultValue)`なら、条件に合う値がないときの値を指定できる。
  購読解除だけでは、`first()`へ完了通知が届くわけではない。

  ```ts
  source$.pipe(first(), takeUntilDestroyed(destroyRef));
  // 破棄時はfirstの上流を購読解除する。

  source$.pipe(takeUntilDestroyed(destroyRef), first());
  // 未取得のまま破棄すると、firstに完了が届きEmptyErrorになり得る。
  ```

### 終了・キャンセル・後片付け

  - `takeWhile(condition)`：条件が成立する間だけ流す。外れた値も出すなら第2引数を`true`にする。
  - `takeUntil(stop$)`：停止用Observableが値を流したら終了。値を出さず完了しただけでは止まらない。
  - `takeUntilDestroyed(destroyRef)`：指定したComponent／Serviceの破棄で購読を終える。
  - `finalize(fn)`：完了・エラー・購読解除で、その購読の後片付けを行う。

  `takeUntilDestroyed()`はinjection context内なら引数を省略できる。
  メソッドやライフサイクルフック内で使うときは、先に取得した`DestroyRef`を渡す。
  `switchMap()`などの後ろに置くと、内側の長い処理も破棄時に止められる。

  `complete`は正常終了の通知で、結果の存在や業務上の成功までは意味しない。
  ローディング解除は`finalize()`、取得した結果の処理は`next`に置く。
  HttpClientは通常応答後に完了するが、破棄時の通信キャンセルが必要なら終了管理を付ける。

### catchError：エラー後の流れを差し替える

  - `catchError(() => of(fallback))`：代わりの正常値を流す。
  - `catchError(() => EMPTY)`：値を流さず、その処理を終える。
  - `catchError(error => throwError(() => error))`：エラーを再通知する。

  **置く位置で、終わる範囲が変わる。**
  `switchMap()`の内側なら1回の要求を処理し、次の入力を待ち続けられる。
  外側で`of()`や`EMPTY`へ差し替えると、入力を待つ流れ全体が終了する。

### awaitする：firstValueFromとlastValueFrom

  - `firstValueFrom()`：最初の値でPromiseを解決し、購読を解除する。
  - `lastValueFrom()`：完了まで待ち、最後の値でPromiseを解決する。
  - 値がないまま完了すると、デフォルト値がなければどちらも`EmptyError`。

  単発の取得を`await`で書きたいときに使う。
  `lastValueFrom()`へ終わらないObservableを渡すと待ち続ける。
  `firstValueFrom()`も、値も完了も来なければ待ち続ける。
  継続的な変更やキャンセルを扱うなら、RxJSのままつなぐ方が自然。

### ページ取得・ポーリング・再試行

  - `expand()`＋`reduce()`：次ページを取り、完了後に全件を1つへまとめる。
  - 次ページがなければ`EMPTY`を返して止める。`reduce()`の結果は完了時に出る。
  - `timer()`＋`exhaustMap()`：定期取得中に次の時刻が来ても、重ねて要求しない。
  - `takeWhile()`：取得結果の状態でポーリングを終了する。
  - `takeUntil(actions$.pipe(ofType(pageLeft)))`：画面離脱Actionで終了する。
  - `retry({ count: 2, delay: 1000 })`：失敗後、1秒待って最大2回再試行する。

  ポーリングでは、時間を作る`timer()`と、APIを呼ぶ処理を分けて読む。
  `retryWhen()`は非推奨。新しく書くなら`retry()`の`delay`設定を使う。

## 6. フォーム・ルーター・HTTP通信

### FormControlとCVA：独自入力をFormsへつなぐ

  `FormControl`は値・検証結果・操作済み・無効状態を持つ。
  `ControlValueAccessor`（CVA）は、それらと独自の入力部品をつなぐ窓口。

  ```html
  <app-custom-input [formControl]="nameControl" />
  ```

  この`[formControl]`を受けるのはAngular Formsの`FormControlDirective`。
  子の通常の`input()`へFormControlを直接渡しているわけではない。

| CVAのメソッド・コールバック | 方向と役割 |
| --- | --- |
| `writeValue(value)` | Forms → 子：値を表示する |
| `registerOnChange(fn)` | 子 → Formsへ値を返す関数を受け取る |
| `registerOnTouched(fn)` | 子 → Formsへ操作済みを知らせる関数を受け取る |
| `setDisabledState(disabled)` | Forms → 子：無効状態を反映する |

  子は受け取った関数を保持し、入力変更で`onChange(value)`、blurなどで`onTouched()`を呼ぶ。
  `writeValue()`から`onChange()`を呼ぶと、親から来た値を親へ戻す循環になるので避ける。

  ```ts
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => CustomInput),
    multi: true,
  }]
  ```

  このproviderで、その部品を値の窓口として登録する。
  `implements ControlValueAccessor`は型の実装確認であり、継承や実行時の登録ではない。
  利用側には`ReactiveFormsModule`のimportも必要。

### canDeactivate：画面を離れてよいか判定する

  未保存の編集があるときなどに、遷移を許可するかを決めるRouterのガード。
  ルート変更の判定であり、タブを閉じる操作まで一律に扱うものではない。

### URLに状態を持たせる

  検索・フィルタ・並び順をURLに置くと、再読み込み・共有・戻る操作で復元しやすい。

  ```text
  操作 → URLを書き換える → URLの変更を読む → 条件に応じて再取得
  ```

  これは設計の選択肢。全ての表示状態をURLに置く必要はない。
  同じ条件をURLとStoreへ別々に持たせる場合は、どちらを正とするか決める。

### HttpClient・fromFetch・Interceptor

  - `HttpClient`：Angularの通常の通信。DI・JSON処理・Interceptor・HTTPテストとつながる。
  - `fromFetch()`：FetchをObservableで扱う。`rxjs/fetch`からimportする。
  - Fetchは404・500でもResponseを返すため、`response.ok`の確認と本文の読み取りが必要。

  `HttpInterceptorFn`は、認証ヘッダーやエラー処理などを通信に共通適用する関数。
  `provideHttpClient(withInterceptors([...]))`で登録する。モック専用ではない。
  登録したHttpClientの通信経路が対象で、直接のFetchには適用されない。

  サーバのデータを必ずStore経由にする決まりはない。
  小さな画面はServiceやResourceでも扱える。共有範囲・責務・既存の構成で決める。

### BFF：画面向けにAPIをまとめる層

  Backend for Frontend。画面と既存APIの間で、データの集約・整形を行う。
  例：複数APIの結果を、一覧表示で使いやすい1つの応答にまとめる。

## 7. ライフサイクルと描画

### どの初期化が終わったのかを区別する

| タイミング | 使う場面 |
| --- | --- |
| `constructor` | インスタンス生成。DI・初期状態の用意。Viewは未初期化 |
| `ngOnChanges` | inputの変更への対応。初回は`ngOnInit`より先 |
| `ngOnInit` | 初期inputが設定された後の初期化。1回 |
| `ngAfterContentInit` | 呼び出し側から渡された内容の初期化後。1回 |
| `ngAfterViewInit` | 自分のViewと子Viewの初期化後。1回 |
| `ngOnDestroy` | 破棄前の後片付け |
| `afterNextRender()` | AngularがDOMへ描画した後に1回 |
| `afterRenderEffect()` | 依存Signalに応じ、DOM描画後に処理 |

  `contentChild()`を初期化後に見るならContent、`viewChild()`ならViewが基準。
  初期化後でも、条件で非表示の子は取得できない。
  `ngAfterViewInit`は毎回の画面更新では呼ばれず、queryの結果はその後も変わり得る。

  DOMサイズの計測や外部UIとの同期は、描画後のAPIを使う。
  `afterNextRender()`などはinjection contextで登録する。サーバ描画時には実行されない。
  寸法の読み取りとDOMへの書き込みを分けると、不要な再計算を減らせる。

## 8. JavaScript・TypeScript

### 演算子：何を「未設定」とみなすか

| 書き方 | 意味 |
| --- | --- |
| `a ?? b` | aが`null`／`undefined`ならb。それ以外はa |
| `a \|\| b` | aがfalsyならb。それ以外はa |
| `a && b` | aがtruthyならb。それ以外はa |
| `a ??= b` | aが`null`／`undefined`のときだけbを代入 |
| `a \|\|= b` | aがfalsyのときだけbを代入 |
| `a &&= b` | aがtruthyのときだけbを代入 |
| `a?.b` | aが`null`／`undefined`なら`undefined`で止める |
| `a?.()` | aが`null`／`undefined`なら呼ばない。それ以外は関数として呼ぶ |
| `a ? b : c` | aがtruthyならb、それ以外はc |
| `!!a` | truthy／falsyを`true`／`false`へ変換 |
| `===`／`!==` | 型を変換せず比較する |
| `...a` | 配列・オブジェクトなどの中身を展開する |
| `...args` | 関数の引数などをまとめて受け取る |
| `const { a, b } = obj` | プロパティを取り出す分割代入 |
| `const [a, b] = arr` | 要素を取り出す分割代入 |
| `x => x.id` | idを返すアロー関数 |
| `x instanceof C` | Cのprototypeがxの継承経路にあるか |

  falsyは`false`・`0`・`-0`・`0n`・`''`・`NaN`・`null`・`undefined`など。
  `0`や空文字を有効な値として残したいなら、初期値の補完には`??`を使う。
  `&&`・`||`は真偽値に限らず、選ばれた元の値を返す。

  ```js
  0 ?? 10;      // 0
  0 || 10;      // 10
  0 && 'OK';    // 0
  ```

  `callback?.()`は関数かどうかを判定しない。関数以外が入る可能性があるなら、次の確認を使う。

  ```ts
  if (typeof callback === 'function') {
    callback();
  }
  ```

### inとincludes：キーか、値か

| 書き方 | 調べるもの |
| --- | --- |
| `'name' in obj` | nameというプロパティがあるか。継承したものも含む |
| `Object.hasOwn(obj, 'name')` | obj自身にnameプロパティがあるか |
| `array.includes('B')` | 配列に値Bがあるか |
| `text.includes('ab')` | 文字列にabが含まれるか |

  `['A', 'B']`に対して、`1 in array`は添字1の有無、`array.includes('B')`は値Bの有無。
  `in`は右辺がオブジェクトである必要がある。

### ユーティリティ型：何を選ぶ・変えるか

| 型 | 効果 |
| --- | --- |
| `Partial<T>` | 全プロパティを任意にする |
| `Required<T>` | 全プロパティを必須にする |
| `Readonly<T>` | 全プロパティを読み取り専用にする |
| `Pick<T, K>` | 指定したプロパティだけ残す |
| `Omit<T, K>` | 指定したプロパティを除く |
| `Record<K, V>` | Kをキー、Vを値の型としてオブジェクト型を作る |
| `Exclude<T, U>` | Union型Tから、Uへ代入可能な候補を除く |
| `Extract<T, U>` | Union型Tから、Uへ代入可能な候補だけ残す |

  - `Partial`・`Required`・`Readonly`：プロパティの性質を変える。
  - `Pick`・`Omit`：オブジェクトのプロパティを選ぶ。
  - `Exclude`・`Extract`：Unionの型の候補を選ぶ。

  ```ts
  type Item = { id: number; title: string; done: boolean };
  type Keys = 'id' | 'title' | 'done';
  type Editable = Exclude<Keys, 'id'>; // 'title' | 'done'
  type Preview = Pick<Item, 'id' | 'title'>;
  ```

  型の変換であり、実行時のデータは変えない。
  `Partial`・`Readonly`などは浅い変換で、入れ子まで一括変換するものではない。

### uniq：配列の重複を除く

  Lodashの`uniq()`は、配列の重複した値を除く。

  ```ts
  const tags = uniq(items?.flatMap(item => item.tags) ?? []);
  ```

  各項目のタグを平らに並べ、重複を除く。項目が未取得なら空配列を渡す。
  オブジェクトは内容ではなく参照で比較される。idでまとめるなら`uniqBy(items, 'id')`。

## 9. 調査・レビューの依頼

  - 調査：「この操作を起点に、呼び出し先・状態変更・表示更新まで、必要なファイルだけ追って。」
  - 解説：「各処理の目的と、次に見る箇所を短く。コードの読み上げは不要。」
  - レビュー：「仕様と照合し、誤り・失敗時・境界条件を確認。修正後も同じ観点で見直して。」
