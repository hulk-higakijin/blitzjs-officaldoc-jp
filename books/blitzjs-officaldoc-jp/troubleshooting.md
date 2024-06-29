---
id: troubleshooting
title: トラブルシューティング
sidebar_label: トラブルシューティング
---

このドキュメントでは、[VS Code デバッガー](https://code.visualstudio.com/docs/editor/debugging) または [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools) を使用して、Blitz のフロントエンドおよびバックエンドコードをフルソースマップのサポートでデバッグする方法について説明します。

Blitz アプリケーションのデバッグには、Node.js にアタッチできる任意のデバッガーを使用できます。詳細は Node.js の [デバッグガイド](https://nodejs.org/en/docs/guides/debugging-getting-started/) を参照してください。

## VS Code でのデバッグ {#vscode}

プロジェクトのルートに `.vscode/launch.json` という名前のファイルを次の内容で作成します：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Blitz: サーバーサイドのデバッグ",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev"
    },
    {
      "name": "Blitz: クライアントサイドのデバッグ",
      "type": "pwa-chrome",
      "request": "launch",
      "url": "http://localhost:3000"
    },
    {
      "name": "Blitz: フルスタックのデバッグ",
      "type": "node-terminal",
      "request": "launch",
      "command": "npm run dev",
      "console": "integratedTerminal",
      "serverReadyAction": {
        "pattern": "started server on .+, url: (https?://.+)",
        "uriFormat": "%s",
        "action": "debugWithChrome"
      }
    }
  ]
}
```

`npm run dev` は `yarn dev`（Yarnを使っている場合）や `blitz dev`（グローバルに Blitz がインストールされている場合）に置き換えることができます。また、アプリケーションが開始するポート番号を [変更している場合](./cli-dev) は、`http://localhost:3000` の `3000` を使用しているポートに置き換えてください。

デバッグパネル（Windows/Linux では <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>D</kbd>、macOS では <kbd>⇧</kbd>+<kbd>⌘</kbd>+<kbd>D</kbd>）に移動し、起動構成を選択して <kbd>F5</kbd> を押すか、コマンドパレットから **Debug: Start Debugging** を選択してデバッグセッションを開始します。

## Chrome DevTools でのデバッグ {#chrome-devtools}

### クライアントサイドコード {#chrome-devtools-client}

通常通りに開発サーバーを `blitz dev`、`npm run dev`、または `yarn dev` を実行して起動します。サーバーが起動したら、Chrome で `http://localhost:3000`（または代替 URL）を開きます。次に、Chrome のデベロッパーツール（Windows/Linux では <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>J</kbd>、macOS では <kbd>⌥</kbd>+<kbd>⌘</kbd>+<kbd>I</kbd>）を開き、**Sources** タブに移動します。

これで、クライアントサイドコードが [`debugger`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger) ステートメントに到達するたびに、コード実行が停止し、そのファイルがデバッグエリアに表示されます。Windows/Linux では <kbd>Ctrl</kbd>+<kbd>P</kbd>、macOS では <kbd>⌘</kbd>+<kbd>P</kbd> を押してファイルを検索し、手動でブレークポイントを設定することもできます。ここで検索すると、ソースファイルは `webpack://_N_E/./` というパスから始まることに注意してください。

### サーバーサイドコード {#chrome-devtools-server}

Chrome DevTools でサーバーサイドの Blitz コードをデバッグするには、Node.js プロセスに [`--inspect`](https://nodejs.org/api/cli.html#cli_inspect_host_port) フラグを渡す必要があります：

```bash
NODE_OPTIONS='--inspect' blitz dev
```

`npm run dev` や `yarn dev` を使用している場合（[Getting Started](/docs/getting-started) を参照）には、`package.json` の `dev` スクリプトを更新する必要があります：

```json
"dev": "NODE_OPTIONS='--inspect' blitz dev"
```

`--inspect` フラグを使用して Blitz 開発サーバーを起動すると、次のようになります：

```bash
Debugger listening on ws://127.0.0.1:9229/0cf90313-350d-4466-a748-cd60f4e47c95
For help, see: https://nodejs.org/en/docs/inspector
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
```

> `NODE_OPTIONS='--inspect' npm run dev` または `NODE_OPTIONS='--inspect' yarn dev` を実行すると機能しないことに注意してください。これは、npm/yarn プロセス用のデバッガーと Blitz のデバッガーを同じポートで起動しようとするためです。すると、コンソールには `Starting inspector on 127.0.0.1:9229 failed: address already in use` というようなエラーが表示されます。

サーバーが起動したら、Chrome で新しいタブを開き、`chrome://inspect` を訪問します。そこでは **Remote Target** セクションに Blitz アプリケーションが表示されます。アプリケーションの下にある **inspect** をクリックして別の DevTools ウィンドウを開き、**Sources** タブに移動します。

ここでのサーバーサイドコードのデバッグは、Chrome DevTools でのクライアントサイドコードのデバッグとほぼ同じように機能します。ただし、<kbd>Ctrl</kbd>+<kbd>P</kbd> または <kbd>⌘</kbd>+<kbd>P</kbd> でファイルを検索する際には、ソースファイルは `webpack://APPLICATION_NAME/./` というパスから始まることに注意してください（`APPLICATION_NAME` は `package.json` ファイルに基づいてアプリケーション名に置き換えられます）。

## デバッグログ {#debug-logging}

デバッグ目的で、Blitz モジュールの詳細なログを有効にする必要がある場合、DEBUG 環境変数を使用して高度なログを有効にすることができます。

Blitz は内部的に [debug](https://www.npmjs.com/package/debug) パッケージを使用しています。

例えば、REPL ログを有効にしたい場合、次のようにコンソールを実行します：

```
DEBUG=blitz:repl blitz console
```

ワイルドカードパターンを使用することもできます：

```
DEBUG=blitz:* blitz console
```

Blitz で使用されている名前空間は次の通りです：

- `blitz:approot`
- `blitz:cli`
- `blitz:config`
- `blitz:errorboundary`
- `blitz:generate`
- `blitz:generator`
- `blitz:installer`
- `blitz:manifest`
- `blitz:middleware`
- `blitz:new`
- `blitz:postinstall`
- `blitz:repl`
- `blitz:rpc`
- `blitz:session`
- `blitz:utils`

### ブラウザでの使用 {#usage-in-the-browser}

Blitz のクライアントサイド部分でもこの種のログを取得することもできます。これを行うには、ローカルストレージ変数 `debug` を `blitz:*` または Blitz で使用されている任意の名前空間に設定し、ページをリフレッシュするだけです。

```
localStorage.debug = 'blitz:*'
```

デベロッパーツールのコンソールで詳細なレベルが有効になっていることを確認してください。

## 詳細情報 {#more-info}

JavaScript デバッガーの使い方について詳しく知りたい場合は、以下のドキュメントを参照してください：

- [VS CodeでのNode.jsデバッグ：ブレークポイント](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_breakpoints)
- [Chrome DevTools：JavaScriptをデバッグ](https://developers.google.com/web/tools/chrome-devtools/javascript)
