# ngel

DevTools からコピーした HTML を整形し、要素のツリーをコメントとして付けるツールです。

## セットアップ

Node.js は `ngel/package.json` の `engines.node` を満たすバージョンを使用してください。
依存関係は `x-spt/ngel` 内で管理します。

```bash
cd x-spt/ngel
npm install
```

## HTML ファイルの変換

```bash
npm run ngel -- xxx.html projects
```

第 1 引数は入力 HTML、第 2 引数は任意のラベルです。
入力の相対パスは `x-spt/ngel` を基準に解決します。
結果は `x-spt/ngel/ngel-out/ngel-001-projects-YYYYMMDD-HHMMSS.html` のような名前で保存します。
連番は既存ファイルに続き、日時は日本時間です。ラベルを省略すると `unlabeled` になります。

リポジトリのルートから直接実行する場合:

```bash
node x-spt/ngel/ngel.mjs xxx.html projects
```

既定ではリポジトリの `src` にある Angular コンポーネントの `selector` を参照し、
見つからない `sm-` タグに `[ns]` を付けます。別のプロジェクトの `src` を参照する場合は、
`--src` にフルパスを指定できます。

```bash
npm run ngel -- xxx.html projects --src "/path/to/project/src"
```

`--src` の相対パスも `x-spt/ngel` を基準に解決します。
参照先が存在しない、または読み取れない場合は、すべてのタグで `[ns]` の記載を省略します。

## Chrome からの取得

`--remote-debugging-port=9222` を指定して起動した Chrome に接続できます。
対象ページを Chrome で開いてから、`x-spt/ngel` で実行します。

```bash
npm run capture -- --label projects --page localhost
```

取得結果も既定で `ngel-out` に保存します。`--out-dir <path>` で変更できます。
capture でも `--src "/path/to/project/src"` を指定できます。
接続先を変更する場合は `--cdp <url>` を指定してください。
既存の Chrome に接続するため、Playwright 用ブラウザーのインストールは不要です。
