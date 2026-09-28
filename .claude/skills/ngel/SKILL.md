---
name: ngel
description: Chrome の画面または DevTools からコピーした HTML ファイルを ngel で整形し、要素ツリー付きの HTML をプロジェクトルート基準の ./x-spt/ngel/ に保存する。「/ngel projects --page localhost」「/ngel --file ./capture.html projects」のように、ラベルと取得オプション、または入力 HTML を渡して使う。
allowed-tools: Bash(node x-spt/ngel/ngel-capture.mjs:*), Bash(node x-spt/ngel/ngel.mjs:*), Bash(npm --prefix x-spt/ngel:*), Read, Glob, AskUserQuestion
---

# ngel

引数: `$ARGUMENTS`

## 実行場所とパス

- プロジェクトルート（作業ディレクトリ）で実行する。
- 保存先は **プロジェクトルートからの相対パス `./x-spt/ngel/`**。両モードとも CLI に `--out-dir "$PWD/x-spt/ngel"` を付ける。`ngel-out/` には保存しない。
- スキルに渡された入力 HTML、`--src`、`--raw` の相対パスもプロジェクトルート基準で扱い、絶対パスに変換して CLI に渡す。絶対パスはそのまま使う。
- `--out-dir` はスキルでは受け付けない。指定されたら、保存先が `./x-spt/ngel/` 固定であることを伝え、使い方を案内する。
- パス、ラベル、セレクターなどの値はそれぞれシェル用にクォートし、値をコマンドとして評価しない。
- 依存関係が未導入なら `npm --prefix x-spt/ngel ci` を実行する。Playwright 用ブラウザーのインストールは不要。

## Chrome から取得

`--file` がない場合は `ngel-capture.mjs` を使う。

1. ラベルを位置引数または `--label` で受け取り、CLI には `--label` で渡す。ラベルがない、または両方に指定されている場合は実行せず、使い方を案内する。
2. ラベル以外は次の capture オプションとして渡す。操作オプションの順序はそのまま保ち、ユーザが指定していないオプションは保存先以外に追加しない。
   ```bash
   node x-spt/ngel/ngel-capture.mjs --label 'projects' --page 'localhost' --out-dir "$PWD/x-spt/ngel"
   ```

   | オプション | 意味 |
   | --- | --- |
   | `--page <URLの一部>` | 対象タブを URL の部分一致で選ぶ。省略時や複数一致時は最初の対象タブを使う |
   | `--cdp <URL>` | Chrome DevTools の接続先。省略時は `NGEL_CDP_URL`、localhost:9222、WSL ゲートウェイの順に試す |
   | `--reuse` | 既存タブを操作・取得する。指定しない場合は新規タブを開く |
   | `--keep` | 取得用に開いた新規タブを残す |
   | `--goto <URLまたはパス>` | 移動する。パスは対象タブの URL 基準 |
   | `--click <selector>` / `--hover <selector>` | 最初に一致する要素をクリック／ホバーする |
   | `--fill <selector> <value>` | 最初に一致する入力欄へ値を入れる |
   | `--press <key>` | `Escape` などのキーを押す |
   | `--wait <msまたはselector>` | 指定時間、または要素の表示を待つ |
   | `--select <selector>` | HTML 全体ではなく、最初に一致する要素だけ取得する |
   | `--src <path>` | Angular の selector を参照するソースディレクトリ。既定はこのリポジトリの `src` |
   | `--settle <ms>` / `--timeout <ms>` | 取得前の待機時間／操作ごとのタイムアウト。既定は 500／15000 |
   | `--raw <path>` | 整形前の HTML も指定ファイルに保存する |

   - `--reuse` がない場合、新規タブへの再表示なので、既存タブで開いたメニューなどの一時的な画面状態は引き継がれない。現在の状態を取得する指示なら `--reuse` を使うことを案内する。
3. 終了コードに応じて結果を伝える。

   | 終了コード | 対応 |
   | --- | --- |
   | 0 | stdout の保存パスをプロジェクトルートからの相対パスにして伝える。生成 HTML は読まず、要約もしない |
   | 2 | 接続エラーを伝える。Chrome の `--remote-debugging-port=9222`、または `--cdp` の指定を案内する |
   | 3 | 引数エラーを伝え、使い方を案内する |
   | 4 | 対象ページのエラーを伝え、対象ページを開くか `--page` を修正するよう案内する |
   | 1 | 実行エラーをそのまま伝える |

## HTML ファイルを変換

`--file <HTMLパス>` がある場合は `ngel.mjs` を使う。

- `--file` の値を CLI の第 1 引数に、任意のラベルを第 2 引数に渡す。ラベル省略時は `unlabeled` になる。
- このモードで受け付けるオプションは `--src` のみ。capture オプションが混ざっていたら、実行せずに使い方を案内する。
- 入力パスと `--src` は前述のとおり絶対パスに変換する。
  ```bash
  node x-spt/ngel/ngel.mjs "$PWD/capture.html" 'projects' --out-dir "$PWD/x-spt/ngel"
  ```
- 成功時は stdout の保存パスをプロジェクトルートからの相対パスにして伝える。生成 HTML は読まず、要約もしない。失敗時はエラーをそのまま伝える。

## 使い方（案内用）

```text
/ngel ラベル [capture オプション]
/ngel --label ラベル [capture オプション]
/ngel --file HTMLパス [ラベル] [--src パス]

/ngel projects --page localhost
/ngel projects --page localhost --reuse --select 'sm-project-list'
/ngel projects --page localhost --goto /projects --click '[data-id="menu"]'
/ngel --file ./capture.html projects
```

保存ファイル名は `./x-spt/ngel/ngel-{連番}-{ラベル}-{YYYYMMDD-HHMMSS}.html`。連番は保存先にある既存の `ngel-数字-*.html` に続き、日時は日本時間。
