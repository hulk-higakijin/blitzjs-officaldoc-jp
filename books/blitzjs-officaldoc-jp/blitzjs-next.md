---
title: "@blitzjs/next"
---

## 概要 {#blitzjs-next-overview}

`@blitzjs/next`アダプターは、Next.jsフレームワーク特有の機能やコンポーネントを提供します。

## セットアップ {#setup-blitzjs-next}

次のコマンドを実行して`@blitzjs/next`をインストールできます:

```bash
npm i @blitzjs/next # yarn add @blitzjs/next # pnpm add @blitzjs/next
```

### Next Config {#blitzjs-next-config}

Blitz.jsは、`next.config.js`ファイルに`blitz`プロパティを受け入れることで拡張します。

```ts
blitz?: {
  resolverPath?: ResolverPathOptions;
  customServer?: {
      hotReload?: boolean;
  };
};
```

:::alert
  カスタム`resolverPath`の設定に関する詳細は、[RPC Specification](/docs/rpc-specification#url)を参照してください。
:::

## クライアント {#blitzjs-next-client}

### 例 {#blitzjs-next-client-example}

`src/blitz-client.ts`内:

```ts
import { setupBlitzClient } from "@blitzjs/next"

export const { withBlitz } = setupBlitzClient({
  plugins: [],
})
```

次に、`src/pages/_app.tsx`内で`MyApp`関数を`withBlitz` HOCコンポーネントでラップします。

```ts
import {
  ErrorFallbackProps,
  ErrorComponent,
  ErrorBoundary,
} from "@blitzjs/next"
import { AuthenticationError, AuthorizationError } from "blitz"
import type { AppProps } from "next/app"
import React, { Suspense } from "react"
import { withBlitz } from "src/blitz-client"

function RootErrorFallback({ error }: ErrorFallbackProps) {
  if (error instanceof AuthenticationError) {
    return <div>Error: You are not authenticated</div>
  } else if (error instanceof AuthorizationError) {
    return (
      <ErrorComponent
        statusCode={error.statusCode}
        title="Sorry, you are not authorized to access this"
      />
    )
  } else {
    return (
      <ErrorComponent
        statusCode={(error as any)?.statusCode || 400}
        title={error.message || error.name}
      />
    )
  }
}

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <ErrorBoundary FallbackComponent={RootErrorFallback}>
      <Component {...pageProps} />
    </ErrorBoundary>
  )
}

export default withBlitz(MyApp)
```

:::alert
  `<ErrorBoundary />`プロバイダーと`<ErrorComponent />`コンポーネントは`@blitzjs/next`により提供されます。
:::

### API {#blitzjs-next-client-api}

```ts
setupBlitzClient({
  plugins: [],
})
```

#### 引数

- `plugins:` Blitz.jsプラグインの配列
  - **必須**

#### 戻り値

`withBlitz` HOCラッパーを含むオブジェクト

## サーバー {#blitzjs-next-server}

### 例 {#blitzjs-next-server-example}

`src/blitz-server.ts`内

```ts
import { setupBlitzServer } from "@blitzjs/next"

export const { gSSP, gSP, api } = setupBlitzServer({
  plugins: [],
})
```

### API {#blitzjs-next-server-api}

```ts
setupBlitzServer({
  plugins: [],
  onError?: (err) => void
})
```

#### 引数

- `plugins:` Blitz.jsプラグインの配列
  - **必須**
- `onError:` エラーをキャッチする_(Sentryなどのサービスに最適)_

#### 戻り値

`[`gSSP`](#blitzjs-next-gssp)`, `[`gSP`](#blitzjs-next-gsp)`, `[`api`](#blitzjs-next-api)`ラッパーを含むオブジェクト。

### カスタムサーバー {#custom-next-server}

Blitz CLIはカスタムNext.jsサーバーの実行をサポートします。これにより、Blitz.js CLIを使用して環境変数を注入しながら、JavaScriptとTypeScriptの両方をコンパイルできます。デフォルトでは、CLIは`src/server/index.[ts | js]`または`src/server.[ts | js]`をチェックします。

カスタムNext.jsサーバーに関する詳細は、[`公式ドキュメント`](https://nextjs.org/docs/advanced-features/custom-server)を参照してください。

## ラッパー {#blitzjs-next-server-wrappers}

すべてのNext.jsラッパー関数は[`superjson`](https://github.com/blitz-js/superjson)でシリアライズされます。これにより、データを返す際に`Date`、`Map`、`Set`、`BigInt`を使用できます。もう一つ注意すべき点は、BlitzはNext.jsリクエストハンドラーを呼び出す前にプラグインのミドルウェアを実行することです。

:::alert
  `gSSP`、`gSP`、`api`関数は、認証プラグインを使用している場合、セッションのコンテキストを引き継ぎます。認証プラグインについての詳細は[@blitzjs/auth](/docs/auth)を参照してください。
:::

### 例 {#blitzjs-next-wrapper-examples}

#### getStaticProps

```ts
import { gSP } from "src/blitz-server"

export const getStaticProps = gSP(async ({ ctx }) => {
  return {
    props: {
      data: {
        userId: ctx?.session.userId,
        session: {
          id: ctx?.session.userId,
          publicData: ctx?.session.$publicData,
        },
      },
    },
  }
})
```

#### getServerSideProps

```ts
import { gSSP } from "src/blitz-server"

export const getServerSideProps = gSSP(async ({ ctx }) => {
  return {
    props: {
      userId: ctx?.session.userId,
      publicData: ctx?.session.$publicData,
    },
  }
})
```

#### api

```ts
import { api } from "src/blitz-server"

export default api(async (req, res, ctx) => {
  res.status(200).json({ userId: ctx?.session.userId })
})
```

_Next.js API routesに関する詳細は、[https://nextjs.org/docs/api-routes/introduction](https://nextjs.org/docs/api-routes/introduction)でドキュメントを参照してください_

## コンセプト {#blitzjs-next-concepts}

### ページ読み込み前にユーザーを認証する {#authenticate-users-on-page-load}

ページが読み込まれる前にユーザーがログインしているかどうかを確認したい場合があります。`getCurrentUser`クエリを`getServerSideProps()`内で使用してクエリを直接呼び出します。これにより、サーバーでユーザーがログインしているかどうかを確認し、Next.jsの組み込みリダイレクトプロパティを使用できます。

```ts
import { Routes, BlitzPage } from "@blitzjs/next"
import { gSSP } from "src/blitz-server"
import getCurrentUser from "src/users/queries/getCurrentUser"

export const getServerSideProps = gSSP(async ({ ctx }) => {
  const currentUser = await getCurrentUser(null, ctx)

  if (currentUser) {
    return {
      props: {
        user: currentUser,
      },
    }
  } else {
    return {
      redirect: {
        destination: Routes.LoginPage(),
        permanent: false,
      },
    }
  }
})
```

### サーバーでのデータフェッチ時の戻り型 {#data-fetching-types}

Next.jsのデータフェッチ関数から返される型を設定できます。Next.js関数のすべてのBlitz.jsラッパーはジェネリックを受け入れます。同様に、`BlitzPage`型も受け入れます。

例えば、TypeScriptユーティリティを使用して`getCurrentUser()`から返される型を取得するのに役立てることができます。

```ts
import { Routes, BlitzPage } from "@blitzjs/next"
import { gSSP } from "src/blitz-server"
import getCurrentUser from "src/users/queries/getCurrentUser"

type TCurrentUser = Awaited<ReturnType<typeof getCurrentUser>>

export const getServerSideProps = gSSP<{ user: TCurrentUser }>(
  async ({ ctx }) => {
    const currentUser = await getCurrentUser(null, ctx)

    if (currentUser) {
      return {
        props: {
          user: currentUser,
        },
      }
    } else {
      return {
        redirect: {
          destination: Routes.LoginPage(),
          permanent: false,
        },
      }
    }
  }
)

const Page: BlitzPage<{ user: TCurrentUser }> = ({ user }) => {
  return (
    <Layout title="Page">
      <div className="container">
        <p>User Page</p>
        {user && <p>{user.email}</p>}
      </div>
    </Layout>
  )
}

export default Page
```

### 初期ページ読み込み時のエラー処理 {#handle-errors-on-load}

クエリでエラーをスローして初期ページ読み込み時にサーバーエラーが発生するエッジケースがあります。この場合、ページが初期的にロードされる際に`_app.tsx`がマウントされるまで`<ErrorBoundary />`がレンダリングされません。これは期待される動作ですが、回避策があります。

たとえば、ユーザーが見つからないクエリでは、`NotFoundError()`を作成してステータスコードを返すことができます。

```ts
export default resolver.pipe(resolver.zod(GetUser), async (input) => {
  const { id } = input

  const user = await db.user.findFirst({ where: { id } })

  if (!user) {
    const userError = new NotFoundError("User not found")
    return {
      error: userError.statusCode,
    }
  } else {
    return {
      user,
    }
  }
})
```

次に、サーバー（この場合は`getServerSideProps()`）でクエリを呼び出し、返されるオブジェクトにエラーキーが含まれている場合はエラーを表示します。

```ts
export const getServerSideProps = gSSP(async ({ ctx }) => {

  const user = await getUser({ 1 }, ctx)
  if("error" in user) {
    return { props: { error: user.error}}
  } else {
    return { props: { user }}
  }
})
```

また、`_app.tsx`でサーバーエラーをキャッチし、トーストコンポーネントでエラーを表示することもできます。

```tsx
function MyApp({ Component, pageProps }: AppProps) {
  const getLayout = Component.getLayout || ((page) => page)
  if (pageProps.error) {
    return <ToastComponent>{pageProps.error.statusCode}</ToastComponent>
  }
  return (
    <ErrorBoundary FallbackComponent={RootErrorFallback}>
      {getLayout(<Component {...pageProps} />)}
    </ErrorBoundary>
  )
}
```
