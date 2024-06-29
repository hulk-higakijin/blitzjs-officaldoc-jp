---
title: ロギング
sidebar_label: ロギング
---

Blitzはデフォルトのロガーとして[tslog](https://tslog.js.org)を使用しています。これを設定するには、`app/blitz-server.ts`ファイルで`logger`プロパティを変更します。

新しいBlitzアプリでは、次のように`logger`プロパティが設定されています。

```ts
// app/blitz-server.ts

import { setupBlitzServer } from "@blitzjs/next"
import { BlitzLogger } from "blitz"

const { gSSP, gSP, api } = setupBlitzServer({
  logger: BlitzLogger({}),
})
```

ロガーの設定には、任意の[tslogオプション](https://tslog.js.org/#/?id=settings)を渡すことができます。

例えば、以下のように設定します：

```ts
// app/blitz-server.ts

import { setupBlitzServer } from "@blitzjs/next"
import { BlitzLogger } from "blitz"

const { gSSP, gSP, api } = setupBlitzServer({
  logger: BlitzLogger({
    colorizePrettyLogs: true,
    prefix: ["[blitz]"],
  }),
})
```

この設定は、RPCやAuthなどのBlitzパッケージで使用されます。

## Blitz RPCロギングの設定 {#blitz-rpc-logging}

`app/blitz-server.ts`で設定された`logger`は、Blitz RPCによって使用され、強力なロギングを提供します。RPCの冗長性、入力と出力のログ記録、および特定のRPCエンドポイントのフィルタリングに関する[追加のロギングオプション](https://blitzjs-com-git-siddhsuresh-blitz-rpc-verbose-blitzjs.vercel.app/docs/rpc-config#blitz-rpc-logging)があります。
