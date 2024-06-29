---
title: 使用ガイド - Next.js 13とBlitzツールキット
---

### 移行ガイド {#migration-guide}

Blitzは、`Next 12` ➔ `Next 13`へのメジャーアップグレードに際して、`pages`ディレクトリの使用では**破壊的変更を必要としません**。全てのファイルを`app`に置いた以前のBlitzレイアウトを引き続き使用することもできますが、新しいNext.jsのアプリルーターを使用したい場合はそのディレクトリの名前を変更する必要があります。

Next.jsチームが提供する移行ガイドに従って、Next 12アプリケーションを完全にアップグレードすることができます。

#### アップグレードガイド {#upgrading-guide}

https://nextjs.org/docs/upgrading#upgrading-from-12-to-13

#### コードモッド {#nextjs-codemods}

https://nextjs.org/docs/advanced-features/codemods

### アプリルーターのサポート（ベータ版） {#app-directory}

[React サーバー サイド コンポーネント](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md)を使用する新しいNextのパラダイムに対応するために、以下のメソッドとフックが新しい`app`ディレクトリで動作するように実装されています。

#### 必要な変更 {#required-changes}

以下のファイルに新しい`use client`ディレクティブを追加します：

1. `src/blitz-client.(ts|js)`
2. `useQuery`, `useInfiniteQuery`, `usePaginatedQuery`, `useMutation`, `Hydrate`、その他Reactクエリクライアントサイドフックを使用する全てのファイル。

**変更しないもの:**

`/pages/api/rpc/[[...blitz]].ts`は`/pages`フォルダ内に残ります。

#### 推奨事項

- ページが`/src/pages`にあるBlitzアプリの場合、アプリディレクトリは`/src/app`に置くことを推奨します。ただし、ルート`/app`のアプリディレクトリもサポートされています。新しいNextJSファイル構造については["プロジェクト組織とファイルの配置"](https://nextjs.org/docs/app/building-your-application/routing/colocation#src-directory)をご覧ください。

#### BlitzProvider {#blitz-provider}

このプロバイダーはアプリ全体をラップし、`(root)/layout.ts`ファイルに配置する必要があります。

**セットアップ**

```ts
// src/blitz-client.ts

"use client"
import {AuthClientPlugin} from "@blitzjs/auth"
import {setupBlitzClient} from "@blitzjs/next"
import {BlitzRpcPlugin} from "@blitzjs/rpc"
import { authConfig } from './blitz-auth-config'

export const {withBlitz, BlitzProvider} = setupBlitzClient({
  plugins: [
    AuthClientPlugin(authConfig),
    BlitzRpcPlugin({}),
  ],
})
```

`authConfig`は、`blitz-client.ts`および`blitz-server.ts`にインポートされる別ファイルにする必要があります：

```ts
// src/blitz-auth-config.ts
import { AuthPluginClientOptions } from '@blitzjs/auth'

export const authConfig: AuthPluginClientOptions = {
  cookiePrefix: "blitz-auth-with-next-app",
}
```

**ルートlayout.tsファイル内**

```tsx
// src/layout.ts
import { BlitzProvider } from "src/blitz-client"

export default function RootLayout({children}: {children: React.ReactNode}) {
  return (
    <html lang="ja">
      <head>
        ...
      </head>
      <body>
        <BlitzProvider>
          ...
        </BlitzProvider>
      </body>
    </html>
  )
}
```

#### Blitz認証 {#blitz-auth}

##### getBlitzContext

この関数はサーバーコンポーネントから提供されるクッキーとヘッダーを使用して現在のセッションを返します。

###### API

```ts
getBlitzContext() => Ctx
```

###### 使用方法

Next 13のアプリディレクトリ内のReactサーバーコンポーネントでの使用例

**セットアップ**

```tsx
// src/blitz-server.ts
export const { ... , useAuthenticatedBlitzContext} = setupBlitzServer({
  ...
})
```

**RSCページまたはレイアウト内**

```tsx
import {getBlitzContext} from "src/blitz-server"
import getCurrentUser from "src/users/queries/getCurrentUser"

export default async function Home() {
  const ctx = await getBlitzContext()
  const user = await getCurrentUser(null, ctx)
  return(
    <>
        ...
    </>
  )
}
```

##### useAuthenticatedBlitzContext

1. このフックは、Next 13 `app`ディレクトリでReactサーバーコンポーネントと連携するように提供された[**BlitzPageのセキュリティ認証ユーティリティ**](https://blitzjs.com/docs/authorization#secure-your-pages)の代替として実装されています。

2. これは、`page.ts`または`layout.ts`のレイアウト内の任意の非同期サーバーコンポーネントで使用できます。

##### API

```tsx
useAuthenticatedBlitzContext({
  redirectTo,
  redirectAuthenticatedTo,
  role,
}: {
  redirectTo?: string | RouteUrlObject
  redirectAuthenticatedTo?: string | RouteUrlObject | ((ctx: Ctx) => string | RouteUrlObject)
  role?: string | string[]
}): Promise<void>
```

##### 各パラメータの説明

1. `redirectTo`

このパラメータに渡される`URL`または`Route`オブジェクトは、**認証されていない**ユーザー（ログアウトされたユーザー）および`role`パラメータに指定された役割を満たさないユーザーを**リダイレクト**するために使用されます。

2. `role`

このパラメータは文字列としての役割または複数の役割を受け取り、特定のページまたはレイアウトへのユーザーアクセスを**認可**するために使用されます。

3. `redirectAuthenticatedTo`

このパラメータに渡される`URL`または`Route`オブジェクトは、**認証された**（ログイン済みの）ユーザーを**リダイレクト**するために使用されます。

##### 使用方法

Next 13のアプリディレクトリ内のReactサーバーコンポーネントでの使用例

**セットアップ**

```tsx
// src/blitz-server.ts
export const { ... , useAuthenticatedBlitzContext} = setupBlitzServer({
  ...
})
```

**RSCページまたはレイアウト内**

```tsx
import {useAuthenticatedBlitzContext} from "src/blitz-server"
...
await useAuthenticatedBlitzContext({
    redirectTo: "/auth/login",
    role: ["admin"],
    redirectAuthenticatedTo: "/dashboard",
})
```

#### Blitz RPC {#blitz-rpc}

Reactサーバーコンポーネント内でリゾルバを呼び出すために使用するメソッドは以下の通りです。

##### `invoke`メソッドの使用

例えば、`(root)/page.js`ファイル内で現在のユーザーの詳細を確認するためにリゾルバをクエリする必要があるとします。

最初に、`invoke`関数を`blitz-server`ファイルからインポートします。

必要なミドルウェアを実行して効果的に機能させるため、`@blitzjs/rpc`から直接`invoke`をインポートすることはできません。

**セットアップ**

```tsx
// src/blitz-server.ts
import {RpcServerPlugin} from "@blitzjs/rpc"
...
export const {... , invoke} = setupBlitzServer({
  plugins: [
    ...
    RpcServerPlugin({}),
  ]
  ...
})
```

**RSCページまたはレイアウト内**

```tsx
import {invoke} from "src/blitz-server"
import getCurrentUser from "src/users/queries/getCurrentUser"
```

では、invoke関数を使用してリゾルバを直接呼び出すことができます。

```tsx
const user = await invoke(getCurrentUser, null)
```
