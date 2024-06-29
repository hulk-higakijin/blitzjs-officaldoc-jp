---
title: HTTP ミドルウェア
sidebar_label: HTTP ミドルウェア
---

HTTP ミドルウェアは、HTTP プリミティブにアクセスするための [クエリ](./query-resolvers) および [ミューテーション](./mutation-resolvers) のエスケープ ハッチです。ミドルウェアは `res.blitzCtx` パラメータを使用してクエリおよびミューテーションにデータを渡すことができます。

ビジネス ロジックにはミドルウェアを使用すべきではありません。なぜなら、HTTP ミドルウェアは再利用性が低く、テストが難しいためです。すべてのビジネス ロジックはクエリ & ミューテーションにあるべきです。

例えば、認証ミドルウェアはクッキーを設定および読み取り、セッション オブジェクトをコンテキスト オブジェクト経由でクエリおよびミューテーションに渡すことができます。その後、クエリおよびミューテーションは認可の実行にセッション オブジェクトを使用し、必要な他の操作を行います。

## グローバル ミドルウェア {#global-middleware}

グローバル ミドルウェアはすべての Blitz クエリおよびミューテーションに対して実行されます。プラグイン配列でヘルパー関数を使用することで `src/blitz-server` に定義できます。

## `BlitzServerMiddleware` {#blitz-server-middleware}

`BlitzServerMiddleware` は、グローバル サーバー ミドルウェアを追加するために使用できるユーティリティ関数です。リクエスト ミドルウェアを Blitz 互換のサーバー プラグインに変換します。

### 例 {#blitz-server-middleware-example}

```ts
...
import {BlitzServerMiddleware} from "@blitzjs/next"
import {BlitzGuardMiddleware} from "@blitz-guard/core/dist/middleware"
...

export const { gSSP, gSP, api } = setupBlitzServer({
  plugins: [
    AuthServerPlugin({
      ...authConfig,
      storage: PrismaStorage(db),
      isAuthorized: simpleRolesIsAuthorized,
    }),
    BlitzServerMiddleware(BlitzGuardMiddleware({
      excluded: [
        '/api/rpc/signup'
      ]
    }))
  ],
})
```

### API {#blitz-server-middleware-api}

```ts
BlitzServerMiddleware(middleware())
```

#### 引数

- `middleware:` `req`、`res` および `next` の引数を持つリクエスト ミドルウェア関数。
  - **必須**

#### 返り値

`BlitzServerPlugin` を返します。これは `app/blitz-server` セットアップ ファイル内で初期化できます。

## グローバル Ctx タイプ {#global-ctx-type}

`res.blitzCtx` パラメータの型にアクセスするには、`blitz` から `Ctx` をインポートします。

次のファイルをプロジェクトに配置することで、`Ctx` タイプを拡張できます。

```ts
// types.ts
declare module "blitz" {
  export interface Ctx {
    customThing: string | null
  }
}
```

`Ctx` に追加した型は、次のようにクエリおよびミューテーション内で自動的に使用できるようになります。

```ts
import { Ctx } from "blitz"

export default async function getThing(input, ctx: Ctx) {
  // ここでは正確に型付けされている
  ctx.customThing
}
```

## ミドルウェア API {#middleware-api}

この API は基本的に connect/Express ミドルウェアと同じですが、非同期の `next()` 関数（Koa のような）を使用する点が異なります。connect とは異なり、Blitz ミドルウェアでは `return next()` または `await next()` を実行して、Promise を返す必要があります。既存の connect ミドルウェア用アダプタ `connectMiddleware()` の使用方法は以下参照。

```js
import { RequestMiddleware } from "blitz"

const middleware: RequestMiddleware = async (req, res, next) => {
  res.blitzCtx.referer = req.headers.referer
  await next()
  console.log("クエリ/ミドルウェアの結果:", res.blitzResult)
}
```

### 引数 {#arguments}

- `req`: `MiddlewareRequest`
  - インスタンス:
    [http.IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage)、
    さらに以下を含む:
  - `req.cookies` - リクエストによって送信されたクッキーを含むオブジェクト。デフォルトは `{}`
  - `req.query` - クエリ文字列
    [query string](https://en.wikipedia.org/wiki/Query_string) を含むオブジェクト。デフォルトは `{}`
  - `req.body` - `content-type` によって解析されたボディを含むオブジェクト。ボディが送信されなかった場合は `null`
- `res`: `MiddlewareResponse`
  - インスタンス:
    [http.ServerResponse](https://nodejs.org/api/http.html#http_class_http_serverresponse)、
    さらに以下を含む:
  - `res.blitzCtx` - クエリおよびミューテーションに 2 番目の引数として渡されるオブジェクト。これがミドルウェアがアプリに通信する方法です
  - `res.blitzResult` - クエリまたはミューテーションから返された結果。これを読み取る前に、`await next()` を行う必要があります
  - `res.status(code)` - ステータス コードを設定するための関数。`code` は有効な [HTTP ステータス コード](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) である必要があります
  - `res.json(json)` - JSON レスポンスを送信します。 `json` は有効な JSON オブジェクトである必要があります
  - `res.send(body)` - HTTP レスポンスを送信します。 `body` は `string`、`object` または `Buffer` にすることができます
- `next()`: `MiddlewareNext`
  - **必須**: ミドルウェア チェーンを続行するためのコールバック。ミドルウェア内でこれを呼び出す必要があります
  - **必須**: Promise を返すため、`await` するか `return next()` のように返す必要があります。Promise はすべての後続のミドルウェアが完了すると解決されます (Blitz のクエリまたはミューテーションを含む)。

## ミドルウェアとクエリ/ミューテーション リゾルバ間の通信 {#communicating-between-middleware-and-query-mutation-resolvers}

### ミドルウェアからリゾルバへ {#from-middleware-to-resolvers}

ミドルウェアは、データ、オブジェクト、関数など何でも `res.blitzCtx` に追加して、Blitz クエリおよびミューテーションに渡すことができます。実行時に `res.blitzCtx` は 2 番目の引数としてクエリ/ミューテーション ハンドラーに自動的に渡されます。

1. `next()` を呼び出す **前** に `res.blitzCtx` に追加する必要があります。
2. 他のミドルウェアも `res.blitzCtx` を変更できるため、慎重にプログラムする必要があります。

### リゾルバからミドルウェアへ {#from-resolvers-to-middleware}

ミドルウェアがリゾルバからデータ、オブジェクト、関数を受け取る方法は 2 つあります。

1. `res.blitzResult` が、クエリ/ミューテーション リゾルバから返される正確な結果を含みます。ただし、これを読み取る前に `await next()` を行う必要があります。`await next()` は、Blitz の内部ミドルウェアを含むすべての後続のミドルウェアが実行された後に解決されます。
2. コールバックを `res.blitzCtx` を通じてリゾルバに渡し、リゾルバが `ctx.someCallback(someData)` を呼び出してミドルウェアにデータを返すことができます。

## エラー処理 {#error-handling}

#### リクエストのショートサーキット

通常、これはミドルウェアから予想されるタイプのエラーです。ミドルウェアはビジネス ロジックを含むべきではないため、認可エラーをスローすることはありません。通常、ミドルウェアでのエラーは、受信リクエストに本当に問題があることを意味します。

リクエストをショートサーキットする方法は 2 つあります。

1. ミドルウェア内でエラーを `throw` する
2. エラー オブジェクトを `next` に渡す (例: `next(error)`)

#### 通常のクエリ/ミューテーション フローによる処理

もう 1 つの一般的ではない方法は、ミドルウェア エラーをクエリ/ミューテーションに渡し、そこからエラーをスローすることです。

```ts
// ミドルウェア内
res.blitzCtx.error = new Error()

// クエリ/ミューテーション内
if (ctx.error) throw ctx.error
```

## Connect/Express 互換性 {#connect-express-compatibility}

Blitz は、connect ミドルウェアを Blitz ミドルウェアに変換する `connectMiddleware()` 関数を提供します。

```ts
// blitz.config.js
const { connectMiddleware } = require("blitz")
const Cors = require("cors")

const cors = Cors({
  methods: ["GET", "POST", "HEAD", "OPTIONS"],
})

module.exports = {
  middleware: [connectMiddleware(cors)],
}
```

### 引数 {#arguments-1}

- `middleware` - 任意の connect/express ミドルウェア。

## 詳細情報 {#more-information}

RPC API の詳細については、
[RPC API ドキュメント](./rpc-specification) を参照してください。
