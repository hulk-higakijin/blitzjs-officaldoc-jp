---
title: BlitzアプリをBlitz 2.0にアップグレードする
---

既存のBlitz.jsアプリがあり、Blitz 2.0にアップグレードしたい場合は、`@blitzjs/codemod`パッケージを使用できます：

```sh
npx @blitzjs/codemod upgrade-legacy
```

上記のコマンドを実行すると、BlitzアプリはBlitz 2.0にアップグレードされます。codemodに問題が発生した場合はお知らせください。また、以下の手動アップグレードガイドもご覧ください。

## 手動アップグレード手順 {#manual-upgrade-steps}

以下では、codemodによって実行される手順をリストします。これらの手順の一部またはすべてを手動で実行したい場合や、codemodツールに問題が発生した場合に備えて、この手順に従ってアプリをアップグレードできます。

### `blitz.config.ts`ファイルの名前を`next.config.js`に変更する {#rename-blitz-config-ts-file}

設定ファイル内で、`@blitzjs/next`からインポートした`withBlitz`関数で設定をラップする必要があります：

```js
// @ts-check
const { withBlitz } = require("@blitzjs/next")

/**
 * @type {import('@blitzjs/next').BlitzConfig}
 **/
const config = {}

module.exports = withBlitz(config)
```

### `package.json`の依存関係を更新する {#update-dependencies}

1. `react`、`react-dom`を最新バージョンに更新します。
2. `@blitzjs/next`、`@blitzjs/rpc`、`@blitzjs/auth`をインストールします。
3. `blitz`バージョンを`latest`に設定します。
4. `next`を最新バージョンにアップグレードします。

### プロジェクトの名前付きインポートを更新する {#update-imports}

以前は`blitz`パッケージからインポートされていた多くのものについて、新しいパッケージにインポートを更新する必要があります。以下のリストを参考にしてください：

| インポート                       | ソースパッケージ     |
| ----------------------------- | --------------- |
| `NextApiHandler`              | `next`          |
| `NextApiRequest`              | `next`          |
| `NextApiResponse`             | `next`          |
| `GetServerSideProps`          | `next`          |
| `InferGetServerSidePropsType` | `next`          |
| `GetServerSidePropsContext`   | `next`          |
| `useRouterQuery`              | `next/router`   |
| `useRouter`                   | `next/router`   |
| `Router`                      | `next/router`   |
| `Link`                        | `next/link`     |
| `Image`                       | `next/image`    |
| `Script`                      | `next/script`   |
| `Document`                    | `next/document` |
| `DocumentHead`                | `next/document` |
| `Html`                        | `next/document` |
| `Main`                        | `next/document` |
| `Head`                        | `next/head`     |
| `App`                         | `next/app`      |
| `dynamic`                     | `next/dynamic`  |
| `noSSR`                       | `next/dynamic`  |
| `getConfig`                   | `next/config`   |
| `setConfig`                   | `next/config`   |
| `ErrorBoundary`               | `@blitzjs/next` |
| `ErrorFallbackProps`          | `@blitzjs/next` |
| `useParam`                    | `@blitzjs/next` |
| `Routes`                      | `@blitzjs/next` |
| `ErrorComponent`              | `@blitzjs/next` |
| `AppProps`                    | `@blitzjs/next` |
| `BlitzPage`                   | `@blitzjs/next` |
| `BlitzLayout`                 | `@blitzjs/next` |
| `invokeWithMiddleware`        | `@blitzjs/rpc`  |
| `resolver`                    | `@blitzjs/rpc`  |
| `useQuery`                    | `@blitzjs/rpc`  |
| `usePaginatedQuery`           | `@blitzjs/rpc`  |
| `useInfiniteQuery`            | `@blitzjs/rpc`  |
| `useMutation`                 | `@blitzjs/rpc`  |
| `queryClient`                 | `@blitzjs/rpc`  |
| `getQueryKey`                 | `@blitzjs/rpc`  |
| `getInfiniteQueryKey`         | `@blitzjs/rpc`  |
| `invalidateQuery`             | `@blitzjs/rpc`  |
| `setQueryData`                | `@blitzjs/rpc`  |
| `useQueryErrorResetBoundary`  | `@blitzjs/rpc`  |
| `QueryClient`                 | `@blitzjs/rpc`  |
| `dehydrate`                   | `@blitzjs/rpc`  |
| `invoke`                      | `@blitzjs/rpc`  |
| `getAntiCSRFToken`            | `@blitzjs/auth` |
| `passportAuth`                | `@blitzjs/auth` |
| `sessionMiddleware`           | `@blitzjs/auth` |
| `simpleRolesIsAuthorized`     | `@blitzjs/auth` |
| `getSession`                  | `@blitzjs/auth` |
| `setPublicDataForUser`        | `@blitzjs/auth` |
| `SecurePassword`              | `@blitzjs/auth` |
| `hash256`                     | `@blitzjs/auth` |
| `generateToken`               | `@blitzjs/auth` |
| `useAuthenticatedSession`     | `@blitzjs/auth` |
| `useRedirectAuthenticated`    | `@blitzjs/auth` |
| `useSession`                  | `@blitzjs/auth` |
| `useAuthorize`                | `@blitzjs/auth` |
| `AuthenticatedSessionContext` | `@blitzjs/auth` |

### プロジェクトのデフォルトインポートを更新する {#update-default-imports}

更新が必要なデフォルトインポートもあります。以下のリストを参考にしてください：

| デフォルトインポート | ソースパッケージ     |
| -------------- | -------------- |
| `Link`         | `next/link`    |
| `Image`        | `next/image`   |
| `Head`         | `next/head`    |
| `dynamic`      | `next/dynamic` |

### `queryClient`のインポートを`getQueryClient`に変更する {#change-query-client-import}

`blitz`からインポートした`queryClient`を次のコードに変更する必要があります：

```diff
- import { queryClient } from 'blitz'

+ import { getQueryClient } from '@blitzjs/rpc'
+ const queryClient = getQueryClient()
```

### `BlitzApiRequest`、`BlitzApiResponse`、`BlitzApiHandler`を更新する {#update-api-imports}

`blitz`は`BlitzApiRequest`、`BlitzApiResponse`、`BlitzApiHandler`をエクスポートしなくなりました。インポートを次のように更新する必要があります：

```diff
- import { BlitzApiRequest } from 'blitz'
+ import { NextApiRequest } from 'next'

- import { BlitzApiResponse } from 'blitz'
+ import { NextApiResponse } from 'next'

- import { BlitzApiHandler } from 'blitz'
+ import { NextApiHandler } from 'next'
```

### `blitz-server.ts`と`blitz-client.ts`を作成する {#create-blitz-files}

プラグインを設定するために、次のファイルをプロジェクトに追加する必要があります：

1. `blitz-server.ts`：

```ts
import { setupBlitzServer } from "@blitzjs/next"
import {
  AuthServerPlugin,
  PrismaStorage,
  simpleRolesIsAuthorized,
} from "@blitzjs/auth"
import { db } from "db"

const { gSSP, gSP, api } = setupBlitzServer({
  plugins: [
    AuthServerPlugin({
      cookiePrefix: "blitz-app",
      storage: PrismaStorage(db),
      isAuthorized: simpleRolesIsAuthorized,
    }),
  ],
})

export { gSSP, gSP, api }
```

2. `blitz-client.ts`：

```ts
import { AuthClientPlugin } from "@blitzjs/auth"
import { setupBlitzClient } from "@blitzjs/next"
import { BlitzRpcPlugin } from "@

blitzjs/rpc"

export const { withBlitz } = setupBlitzClient({
  plugins: [
    AuthClientPlugin({
      cookiePrefix: "blitz-app",
    }),
    BlitzRpcPlugin(),
  ],
})
```

### Zero-APIレイヤーのAPIルートを作成する {#create-api-route}

Zero-APIレイヤーを設定するには、`src/pages/api/rpc/[[...blitz]].ts`ファイルを次の内容で作成する必要があります：

```ts
import { rpcHandler } from "@blitzjs/rpc"
import { api } from "src/blitz-server"

export default api(rpcHandler())
```

### `babel.config.js`を削除する {#remove-babel-config}

もう必要ありません。

### すべてのページを`src/pages`ディレクトリに移動する {#move-pages-to-pages}

Next.jsでは、ページを別々のディレクトリに配置することはサポートされていません。これらは上位レベルの`pages`ディレクトリに統合するか、`src`ディレクトリに配置することをお勧めします。

### APIルートを`src/pages/api`ディレクトリに移動する {#move-api-routes-to-pages-api}

すべてのAPIルートは`src/pages/api`ディレクトリ内にある必要があります。

### Appコンポーネントを`withBlitz`でラップする {#wrap-app-with-blitz}

クライアントでBlitzを使用するには、Appコンポーネント内で`withBlitz`関数を使用する必要があります。

```ts
import { withBlitz } from "src/blitz-client"

function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}

export default withBlitz(App)
```

### `useRouterQuery`を`useRouter`に変換する {#convert-use-router-query-to-use-router}

Blitzは`useRouterQuery`をエクスポートしなくなりました。代わりに`useRouter`関数を使用する必要があります。

```ts
import { useRouter } from "next/router"
```

### `getServerSideProps`、`getStaticProps`、APIルートをラップする {#wrap-get-ssp-static-props-api-routes}

`getServerSideProps`、`getStaticProps`、APIルート内でBlitzコンテキストにアクセスしたい場合は、それらを`@blitzjs/next`からインポートした対応する関数：`gSSP`、`gSP`、`api`でラップする必要があります。

詳細については、[`@blitzjs/next`のドキュメント](./blitzjs-next)を参照してください。

### `queryClient`を`getQueryClient`関数に置き換える {#replace-query-client-with-get-query-client}

コード内で`queryClient`を使用している場合は、次のコードに置き換えることができます：

```ts
import { getQueryClient } from "@blitzjs/rpc"

const queryClient = getQueryClient()
```

### `types.ts`の変更 {#types-changes}

`types.ts`ファイル内で、拡張されるモジュールを次のように変更する必要があります：

```diff
- declare module "@blitzjs/auth" {
+ declare module "@blitzjs/auth" {
    export interface Session {
      // ...
    }
  }
```
