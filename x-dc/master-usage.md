# 開発作業の使い方メモ

よく使う操作・コマンド・依頼文を、やりたいことから探せるようにまとめる。

- [VS Code](#vs-code)
- [ブラウザー・不具合調査](#ブラウザー不具合調査)
- [検索・ファイル操作・比較](#検索ファイル操作比較)
- [Git・GitHub](#gitgithub)
- [Angular のコード調査](#angular-のコード調査)
- [AI への依頼文](#ai-への依頼文)
- [その他の小技](#その他の小技)

## VSCode

### よく使うショートカット

Mac の `Cmd` は ⌘、`Option` は ⌥。キー配列や個別設定で異なる場合は「キーボード ショートカット」で確認する。

| やりたいこと | macOS | Windows / Linux |
| --- | --- | --- |
| ファイルを名前で探す | `Cmd + P` | `Ctrl + P` |
| コマンドパレットを開く | `Cmd + Shift + P` | `Ctrl + Shift + P` |
| 選択範囲を括弧・構文単位で広げる | `Ctrl + Shift + Cmd + →` | `Shift + Alt + →` |
| 表示上の折り返しを切り替える | `Option + Z` | `Alt + Z` |
| 複数箇所にカーソルを置く | `Option + クリック` | `Alt + クリック` |
| 対応する括弧へ移動する | `Cmd + Shift + \` | `Ctrl + Shift + \` |
| 行を丸ごと上下へ移動する | `Option + ↑ / ↓` | `Alt + ↑ / ↓` |
| 選択した文字列と同じものを全部選ぶ | `Cmd + Shift + L` | `Ctrl + Shift + L` |
| 同じ文字列を1個ずつ追加で選ぶ | `Cmd + D` | `Ctrl + D` |
| ファイル内の検索結果を全部選ぶ | `Cmd + F` → `Option + Enter` | `Ctrl + F` → `Alt + Enter` |
| 定義へ移動する | `F12` | `F12` |
| 参照元を探す | `Shift + F12` | `Shift + F12` |

例：`itemName` を選ぶ → 全部選ぶ → 入力すると、同じ文字列を一括で書き換えられる。[公式ショートカット一覧](https://code.visualstudio.com/docs/reference/default-keybindings)

### ターミナルを切り替える

ターミナルにフォーカスがある状態で操作する。

| 対象 | macOS | Windows / Linux |
| --- | --- | --- |
| 次 / 前のターミナルグループ | `Cmd + Shift + ] / [` | `Ctrl + PageDown / PageUp` |
| 同じグループ内の分割ターミナル | `Option + Cmd + → / ←` | `Alt + → / ←` |

コマンド名は `Terminal: Focus Next Terminal` / `Terminal: Focus Previous Terminal`。[公式の説明](https://code.visualstudio.com/docs/terminal/basics#managing-terminals)

### タブ幅・プレビュー表示

設定 JSON の例。プレビューは一時表示で、別のファイルを開くと同じタブが使い回される。

```json
{
  "workbench.editor.tabSizing": "fixed",
  "workbench.editor.tabSizingFixedMinWidth": 120,
  "workbench.editor.tabSizingFixedMaxWidth": 170,
  "workbench.editor.pinnedTabSizing": "shrink",
  "workbench.editor.enablePreview": true,
  "workbench.editor.enablePreviewFromCodeNavigation": false,
  "workbench.editor.enablePreviewFromQuickOpen": true
}
```

この例では、通常のファイルオープンと `Cmd / Ctrl + P` は一時表示、定義への移動は固定表示。タブをダブルクリックすると一時表示を固定できる。

### 左に目次、右にリンク先を表示する

1. エディターを左右に分割し、左に目次用 Markdown を開く。
2. 左の `…` → `Lock Group`。コマンド名は `View: Lock Editor Group`。
3. 右はロックせず、`markdown.links.openLocation` を `beside` にする。

### ファイルをコマンドで開く・拡張機能を使う

```bash
code ~/.gitignore_global
```

- コメントの翻訳：`Comment Translate`。
- AI 拡張にコードを渡す：開いているファイルや選択範囲を会話の文脈に追加する。
- AI 拡張をサイドバーに置く：拡張の表示位置設定やコマンドパレットから切り替える。

### Multi-root Workspace

これを使うと、GitHub Copilot Agent が複数のプロジェクトを横断して、ファイルの参照だけでなく編集もできるようになります。

例えば、以下の構成があるとします。

```
work/
├── aaa-project/       ← VS Codeで開いている
├── bbb-project/
└── ccc-project/
```

| 操作                      | 通常   | Multi-root |
| ----------------------- | ---- | ---------- |
| `aaa-project` の参照・編集    | ○    | ○          |
| `../bbb-project` の参照 | 条件付き | ○          |
| `../bbb-project` の編集 | 制限あり | ○          |
| `../ccc-project` の編集 | 制限あり | ○          |

通常でもワークスペース外への編集が絶対に不可能というわけではありません。エージェントの権限設定や承認によって変わります。

#### 設定方法

VS Codeで次の操作をします。

1. `File` → `Add Folder to Workspace...`
2. `bbb-project` を追加
3. 同じ操作で `ccc-project` も追加
4. `File` → `Save Workspace As...` で保存

これで、例えば以下のワークスペースファイルができます。

`qqq.code-workspace`

```
{
  "folders": [
    { "path": "aaa-project" },
    { "path": "bbb-project" },
    { "path": "ccc-project" }
  ]
}
```

※ この例は `.code-workspace` ファイルを `work` フォルダに保存した場合です。

以降はこのファイルを開くだけで、3プロジェクトをまとめて扱えます。

Copilot Agent に、

「aaa-project と bbb-project の関連処理を調査して、双方に必要な変更を加えて」

- 3つのプロジェクトは、それぞれ独立したGitリポジトリでよい
- 無理にモノレポ化する必要はない
- VS Codeのソース管理にも個別のリポジトリとして表示される
- Copilotは複数のプロジェクトをまとめて解析・編集できる


## ブラウザー・不具合調査

### Chrome DevTools

| やりたいこと | macOS | Windows / Linux |
| --- | --- | --- |
| 画面上の要素を選んで検証する | `Cmd + Shift + C` | `Ctrl + Shift + C` |
| DevTools を開く | `Cmd + Option + I` | `Ctrl + Shift + I` |

要素選択のショートカットは、DevTools を開いた後も選択モードの切り替えに使える。[公式ショートカット一覧](https://developer.chrome.com/docs/devtools/shortcuts)

### 症状から調査場所を選ぶ

| 見たいもの | 最初に見る場所 |
| --- | --- |
| 表示崩れ | DevTools の Elements・CSS |
| JavaScript の例外 | Console |
| API の失敗・送受信内容 | Network |
| API の遅さ | Network の Timing |
| 描画・操作の遅さ | Performance |
| Angular の状態・コンポーネント | Angular DevTools |
| NgRx の Action・State | Redux DevTools |
| コンポーネント・処理のつながり | VS Code の定義・参照検索 |
| Cookie・ブラウザー保存 | Application |
| 認証 | Network・Application・認証処理のコード |
| サーバー・Python の処理 | ログ・デバッガー |
| コンテナ障害 | Docker / Kubernetes のログ |
| DB・キャッシュの内容 | DB / Redis クライアント |

### デバッグ用 Chrome を起動する（macOS）

```bash
open -na "Google Chrome" --args \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/browser-debug-profile
```

普段使うプロファイルと分けて起動する。

リモート側からこのポートを使う場合の SSH 転送例。`user@remote-host` と SSH ポートは接続先に置き換える。

```bash
ssh -v -p 2222 -NT -o ExitOnForwardFailure=yes \
  -R 9222:127.0.0.1:9222 user@remote-host
```

`ExitOnForwardFailure=yes` は、転送用ポートを確保できなければ終了する指定。

## 検索・ファイル操作・比較

### 文字列・ファイル名を探す

```bash
# 内容を検索し、行番号も表示
rg -n '検索したい文字列' src

# 該当するファイル名だけ表示
rg -l '検索したい文字列' src

# ファイル名に item を含むものを探す
rg --files src | rg -i 'item'
```

### Bash の小技

`>` 区切りのセレクターを、1要素ずつ改行する。`.bashrc` などに追加する。

```bash
selspl() {
  printf '%s\n' "$1" | perl -pe 's/ > /\n/g'
}

selspl 'body > app-root > app-item-editor > input'
```

作業ディレクトリを一時的に変えるには、括弧で囲む。実行後は元の場所のまま。

```bash
(cd .. && ls -l)
```

### node_modules を整理する

```bash
npx npkill
```

一覧から削除対象を選べる。必要になった依存関係は再インストールする。

### .git を除いてコピーする

```bash
# 先にコピー内容を確認
rsync -a --dry-run --exclude='.git/' ./source/ ./destination/

# コピーを実行
rsync -a --exclude='.git/' ./source/ ./destination/
```

コピー元の末尾 `/` は「フォルダーの中身」をコピーする指定。同名ファイルはコピー元の内容で更新される。

### フォルダーを比較する

```bash
diff -rq \
  --exclude=node_modules \
  --exclude=.git \
  --exclude=dist \
  --exclude=build \
  --exclude=.angular \
  ./source ./destination

# Git 管理外の2つのフォルダーも比較できる
git diff --no-index --name-status ./source/src ./destination/src
```

### WinMerge のフィルター

比較開始前の「ファイルまたはフォルダーの選択」→「フィルター」に指定する。

```text
!node_modules\;!.git\;!dist\;!build\;!.angular\
```

比較中は「ツール」→「フィルター」で変更し、`F5` で再比較する。[公式の説明](https://manual.winmerge.org/en/Filters.html)

生成日時の行を差分から除く行フィルター例：

```regex
^Generated at:\s*\d{4}-\d{2}-\d{2}\s+\d{2}:\d{2}:\d{2}$
```

### 正規表現

| やりたいこと | 正規表現 |
| --- | --- |
| 同じ行に `abc` と `DEF` を含む | `^(?=.*abc)(?=.*DEF).*$` |
| `abc` を含まない行 | `^(?!.*abc).*$` |
| `app-` で始まる開始タグを探す | `<app-[\w-]+(?=[\s/>])` |

改行削除は `\r\n|\r|\n` を検索し、空文字に置換する。先読みを含む検索は、VS Code の正規表現モードや `rg --pcre2` で使う。

## Git・GitHub

### 履歴を見る・設定を変える

```bash
git log --oneline | less -S
git log --oneline -3

# 日本語のファイル名をそのまま表示
git config core.quotepath false

# 全リポジトリで日本語表示にする場合
git config --global core.quotepath false

# コミットなどに使うエディターを変更
git config --global core.editor vim
```

### .gitignore を変えずに除外する

| 対象 | 記載場所 |
| --- | --- |
| このリポジトリだけ | `.git/info/exclude` |
| 自分の全リポジトリ | `~/.gitignore_global` など、設定で指定したファイル |

```bash
git config --global core.excludesFile ~/.gitignore_global
```

書式は `.gitignore` と同じ。すでに追跡中のファイルには効かない。[公式の説明](https://git-scm.com/docs/gitignore)

### worktree で別ブランチを同時に開く

既存ブランチ `feature/example` を別フォルダーで開く例：

```bash
git worktree add ../feature-work feature/example
git worktree list
```

### fixup：修正を過去のコミットにまとめる

`TARGET` を吸収先のコミット ID に置き換える。

```bash
git add -A
git commit --fixup=TARGET
git rebase -i --autosquash TARGET^
```

途中で中止する場合は `git rebase --abort`。吸収先が最初のコミットなら、最後のコマンドは `git rebase -i --autosquash --root` にする。

### 過去のコミットメッセージを変える

未コミットの変更がない状態で実行する。

```bash
git rebase -i HEAD~3
```

一覧は上から古い順。変更したい行の `pick` を `reword` に変えて保存し、メッセージを修正する。

共有済みの履歴を書き換える場合は共同作業者と調整し、反映には `git push --force-with-lease origin branch-name` を使う。

### リモートブランチを削除する

削除対象を確認し、`branch-name` を置き換える。

```bash
git push origin --delete branch-name
```

### コミットメッセージの接頭辞

| type | 意味 | 例 |
| --- | --- | --- |
| `feat:` | 新機能 | `feat: CSV出力を追加` |
| `fix:` | バグ修正 | `fix: 一覧の表示崩れを修正` |
| `docs:` | 文書 | `docs: 起動手順を追加` |
| `refactor:` | 動作を変えない整理 | `refactor: 重複処理を共通化` |
| `test:` | テスト | `test: サービスのテストを追加` |
| `chore:` | 保守・雑務 | `chore: 不要ファイルを削除` |
| `style:` | 整形 | `style: インデントを修正` |
| `perf:` | 性能改善 | `perf: 一覧取得を高速化` |
| `build:` | ビルド | `build: ビルド設定を変更` |
| `ci:` | CI/CD | `ci: 自動テストを追加` |
| `revert:` | コミット取り消し | `revert: CSV出力の追加を取り消し` |

### コメントだけを Git の差分から除きたい場合

clean filter は表示だけでなくコミットに保存する内容も変える。解説コメントを残したい場合は、機能変更とコメント追加を別コミットに分ける。

### GitHub で再レビューを依頼する

PR の `Conversation` → 右側の `Reviewers` → 対象者横の再レビュー依頼アイコンを選ぶ。

### sub modules

通常のcloneでは、サブモジュールの中身は取得されません。  
git clone <リポジトリURL>

この場合、clone後に次を実行します。  
git submodule update --init --recursive

一方、最初からサブモジュールもまとめて取得するなら、  
git clone --recurse-submodules <リポジトリURL>

1. `git submodule status`  
サブモジュールの一覧と状態を確認。先頭が `-` なら未初期化、`+` なら登録コミットと不一致。

2. `cat .gitmodules`  
サブモジュールのパスと取得先URLを確認。

3. `git ls-files --stage -- {submoduleフォルダ名}`  
`{submoduleフォルダ名}`がサブモジュールとして登録されているか確認。先頭が `160000` ならサブモジュール。

4. `ls -la {submoduleフォルダ名}`  
隠しファイルを含めてフォルダの中身を確認。

5. `git submodule update --init --recursive -- {submoduleフォルダ名}`  
未初期化のサブモジュールを取得。入れ子のサブモジュールも取得する。

6. `git submodule foreach 'git status -sb'`  
初期化済みの各サブモジュールのGit状態を確認。

7. `git submodule update --remote -- {submoduleフォルダ名}`  
リモートの追跡対象ブランチに合わせてサブモジュールを更新。親リポジトリが指定するコミットから変わる可能性があるため、業務環境では注意。

## Angular のコード調査

### コンポーネントや画面操作を追う補助ツール

`ng-maze` / `ng-wiring` を導入済みの場合の例。プロジェクト内にインストールした CLI を実行する。

```bash
# コンポーネント全体のつながり
npx --no ngmaze --all --mdh -p .

# 指定コンポーネントの子・親を調べる
npx --no ngmaze ItemPageComponent --mdh
npx --no ngmaze ItemEditorComponent --parents --mdh --with-routes

# 画面の属性を起点に処理を調べる
npx --no ng-wiring 'data-id="titleInput"' --project app \
  --selector 'body > app-root > app-item-editor > input'
```

`app` は `angular.json` のプロジェクト名に置き換える。実行時に生成される ID より、ソースに書かれた属性を起点にする。候補が複数あれば経路を選び、ダイアログ内で見つからない場合は保存ボタンなど別の要素から追う。

### madge：import の依存関係を調べる

TypeScript では `--extensions ts`、パス別名を解決するために `--ts-config` を指定する。設定ファイルは対象プロジェクトに合わせる。

```bash
# 循環依存
npx madge --circular --extensions ts --ts-config tsconfig.app.json src/app

# 指定ファイルを import しているファイル
npx madge --depends shared/utils/format.ts --extensions ts --ts-config tsconfig.app.json src/app

# 指定ファイルを起点に依存図を保存
npx madge --image deps.svg --extensions ts --ts-config tsconfig.app.json src/app/item/item.component.ts

# JSON で保存
npx madge --json --extensions ts --ts-config tsconfig.app.json src/app > deps.json
```

同じ基本コマンドで使えるオプション：

| オプション | 見られるもの |
| --- | --- |
| `--orphans` | 解析対象内で、他のファイルから import されていないもの |
| `--leaves` | 他のファイルを import していない末端 |
| `--summary` | ファイルごとの依存数 |
| `--dot` | Graphviz 用の DOT 出力 |

import されていないことだけで不要とは判断しない。起点ファイルや動的な利用も確認する。[madge の説明](https://github.com/pahen/madge)

依存図の画像出力や PlantUML の一部の図には Graphviz が必要。Ubuntu での導入例：

```bash
sudo apt install graphviz
```

## AI への依頼文

`___対象___` などを置き換えて使う。

### 調査範囲を絞る

```text
___指示___
___パス___ を起点に、まずその配下だけ調査してください。
外部参照が必要な場合だけ関連ファイルへ広げ、無関係なディレクトリは探索しないでください。
```

### 人間がコードを追える手順にする

```text
___処理___ を、定義への移動・参照検索・文字列検索だけで追える手順にしてください。
各手順には「何を知るために、どのファイルで、何をクリック・検索するか」を書いてください。
子から親への受け渡しも省略せず、主経路と補足を分けてください。
説明は短くし、次の場所へ移るつながりは端折らないでください。
```

例：ダイアログの `(click)="close('save')"` から保存処理を追う場合。

1. `close` の定義へ移動し、閉じるときに渡す値を確認する。
2. ダイアログのクラスの参照元を探し、`dialog.open(EditorDialogComponent)` の呼び出し元を開く。
3. そのダイアログの `afterClosed()` を見て、閉じた後に値を受け取る処理を確認する。
4. 保存分岐から呼ばれる関数へ進む。NgRx の `dispatch` があれば Action を検索する。
5. 対応する Effect → サービス → HTTP 送信へ進む。

### コードに読む順番を付ける

```text
___起点___ から読む順番を、コードの末尾コメントに //:: (1) の形式で付けてください。
HTML は <!-- //:: (1) ... --> の形式にしてください。
各コメントには「何を確認し、次にどこを見るか」を短く書いてください。
主経路の番号を順につなげ、補足は主経路の番号に混ぜないでください。
クリックや検索で次へ移る手順も示してください。
```

```html
<button (click)="close('save')">保存</button> <!-- //:: (1) close の定義へ移動し、閉じるときに渡す値を確認する。 -->
```

### 状態・処理の遷移リストを作る

```text
___起点___ から、HTTP 送信・画面反映・ファイル出力までの遷移リストを作ってください。
各行は「順番・ソースリンク・クラス/関数名・受け渡す値や操作」を短く書いてください。
リンクは [▶️:種類:役割](相対パス#行番号) の形式にしてください。
終点を 01 とし、起点から終点へ番号を減らして並べてください。
関係しないコンポーネントなどを無理に加えないでください。
```

以下は架空の構成例。実際のパス・行番号・処理名に置き換える。

```markdown
- 05. [▶️:C:D](src/app/item-editor/item-editor.component.ts#20) ItemEditorComponent.titleUpdated.emit(title)
- 04. [▶️:h:D](src/app/item-page/item-page.component.html#8) 親が titleUpdated を受けて saveTitle($event) を呼ぶ
- 03. [▶️:D:D](src/app/item-page/item-page.component.ts#30) saveTitle(title) が対象の id を添えて itemUpdated({ id, title }) を dispatch
- 02. [▶️:E:D](src/app/item-state/item.effects.ts#15) ItemEffects.updateItem$ が ItemApi.update(id, title) を呼ぶ
- 01. [▶️:S:A](src/app/item-api/item-api.service.ts#12) ItemApi.update(id, title) が PATCH /items/{id} に { title } を送信
```

| 区分 | 記号の意味 |
| --- | --- |
| 種類 | `h` HTML、`C` コンポーネント、`D` dispatch、`E` Effect、`S` サービス、`?` その他 |
| 役割 | `D` データ受け渡し、`A` API 通信、`-` 該当なし |

dispatch を含む行は種類を `D` にする。行番号付きリンクの動作は閲覧環境による。

### 敵対的レビューと修正を繰り返す

```text
___対象___ に対して敵対的レビューを実施してください。
前提の誤り、見落としたリスク、二次的影響、失敗する条件を確認し、妥当な指摘は即時修正してください。
修正後に再レビューし、最大3回まで直列で繰り返してください。
指摘を無理に作らず、問題がなくなったら終了してください。
設計変更がある場合は、変更後の前提で再評価してください。
```

指摘を資料に残してから修正する場合：

```text
___対象___ の指摘を ___指摘資料___ にまとめてください。
別のレビューで追加された指摘も、そのまま採用せず根拠を再確認し、資料全体を校正してください。
校正した資料に沿って対象を修正し、再レビューしてください。
```

### UI/UX の改善案を出す

```text
___画面___ の実装・挙動を UI/UX の観点でレビューしてください。
迷いにくさ、操作量、待ち時間、エラーからの回復、アクセシビリティを確認してください。
改善案は理由・具体的な変更・優先度を短く書き、内容ごとに ___保存先___ へ資料化してください。
```

### 基礎を短時間で理解する

```text
___テーマ___ を基礎から15分で理解できるように説明してください。
用語を省略せず、具体例でつないでください。子と親の関係がある場合は、その都度受け渡しを説明してください。
```

### 関数名を頭の中で言い換える

「自分なら何という名前を付けるか」で役割をつかむ。以下は理解用の仮名で、実際の API 名ではない。

| 正式 API | 頭の中の仮名 |
| --- | --- |
| `toSignal()` | `observableToSignal()` |
| `resource()` | `asyncLogicToResource()` |
| `rxResource()` | `observableLogicToResource()` |
| `httpResource()` | `httpLogicToResource()` |
| `computed()` | `signalsToDerivedValue()` |
| `linkedSignal()` | `sourceLinkedWritableSignal()` |
| `effect()` | `signalChangeToSideEffect()` |

## その他の小技

- Emmet：`td*5`、`ul>li*5`、`ul>li.item$*5`。[チートシート](https://docs.emmet.io/cheat-sheet/)
- ツリー構造の整形：[Tree](https://tree.nathanfriend.com/)。
- CSV 編集：`SmoothCSV`。
- HTTP リクエストの確認：API クライアントや DevTools の Network。
- Windows のかな / ローマ字入力：`Alt + カタカナ・ひらがな・ローマ字`。
- Windows の一時ファイル：エクスプローラーのアドレス欄に `%TEMP%` を入力する。
