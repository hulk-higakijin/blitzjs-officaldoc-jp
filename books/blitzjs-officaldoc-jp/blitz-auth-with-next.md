---
title: Next.jsでのBlitz認証
---

このガイドでは、Next.jsアプリにBlitz Authを追加する方法をカバーします。BlitzとNext.jsの設定、基本的な認証ロジックの作成、およびBlitz Auth機能をNextのページとAPIハンドラ内で使用する方法について説明します。

ガイドを進める際に詰まった場合は、[このリポジトリ](https://github.com/beerose/next13-blitz-auth)を参照してください。

## 新しいNext.jsアプリの作成 {#create-new-next-app}

まず、`create-next-app`を使用して新しいNextアプリケーションを作成します。

```sh
npx create-next-app@latest
```

次に、`yarn dev`を実行し、ブラウザで`http://localhost:3000`を開いて確認できます。

## Blitzの設定 {#blitz-setup}

### 依存関係のインストール {#install-dependencies}

次に、Blitz.jsのパッケージをインストールする必要があります。

- `blitz` — Blitzのコアパッケージで、Blitz CLI、ユーティリティ、および他のBlitzパッケージで使用されるコア関数を含みます。
- `@blitzjs/next` — Next.jsフレームワークアダプタで、NextプロジェクトでBlitzを初期化するために必要です。
- `@blitzjs/auth` — このガイドで探るAuthプラグインです。

```sh
yarn add blitz @blitzjs/next @blitzjs/auth
```

アプリ内でBlitzプラグインを機能させるためには、クライアント側とサーバー側の2つの設定ファイルが必要です。まずは`src`ディレクトリを作成し、クライアント側から始めましょう。

### `blitz-client.ts`ファイルを追加 {#add-blitz-client}

新しいファイル`blitz-client.ts`を作成し、次の内容を追加します。

```ts
// src/blitz-client.ts:
import { AuthClientPlugin } from "@blitzjs/auth"
import { setupBlitzClient } from "@blitzjs/next"

export const authConfig = {
  cookiePrefix: "blitz-auth-with-next-app",
}

export const { withBlitz } = setupBlitzClient({
  plugins: [AuthClientPlugin(authConfig)],
})
```

ここでは、`@blitzjs/next`の`setupBlitzClient`を使用し、使用したいプラグインの設定を提供します。今回の場合、関心があるのはAuthプラグインだけです。設定する必要があるのはクッキーのプレフィックスだけで、他はBlitzのデフォルトに頼ります。`authConfig`を別の変数として持つ理由は、それをエクスポートしてサーバー設定で再利用できるようにするためです。

### `blitz-server.ts`ファイルを追加 {#add-blitz-server}

次に、サーバー側の設定を行います。同じディレクトリに`blitz-server.ts`ファイルを追加します。

```ts
// src/blitz-server.ts
import { setupBlitzServer } from "@blitzjs/next"
import {
  AuthServerPlugin,
  PrismaStorage,
  simpleRolesIsAuthorized,
} from "@blitzjs/auth"
import { BlitzLogger } from "blitz"
import db from "../prisma"
import { authConfig } from "./blitz-client"

export const { gSSP, gSP, api } = setupBlitzServer({
  plugins: [
    AuthServerPlugin({
      ...authConfig,
      storage: PrismaStorage(db),
      isAuthorized: simpleRolesIsAuthorized,
    }),
  ],
})
```

このファイルでは、以下のようなことが行われています。

- `@blitzjs/next`の`setupBlitzServer`を使用（クライアント設定ファイルと同様）
- Authプラグインのサーバー側設定では、2つの追加の要素が必要です。
  - `storage` : Blitzはセッション情報をデータベースに保存するため、Blitz Authがストレージにアクセスする方法を指定する必要があります。
  - `isAuthorized`: ページや他の保護されたコードにアクセスするユーザーロールを決定する関数です。Blitz Authが提供する`simpleRolesIsAuthorized`を使用します。詳細はこちらを参照してください：https://blitzjs.com/docs/authorization#is-authorized-adapters。

### `types.ts`ファイルを追加 {#add-types}

`blitz-server.ts`ファイルを作成した後、Blitz Auth設定の`storage`プロパティに関連するTypeScriptの問題が発生するかもしれません。これは、Blitzが型拡張を使用して`Session`オブジェクトの見た目を設定するためです。プロジェクトのルートに`types.ts`ファイルを作成します。

```ts
import { SimpleRolesIsAuthorized } from "@blitzjs/auth"
import { User } from "./prisma"

export type Role = "ADMIN" | "USER"

declare module "@blitzjs/auth" {
  export interface Session {
    isAuthorized: SimpleRolesIsAuthorized<Role>
    PublicData: {
      userId: User["id"]
      email: string
    }
  }
}
```

### データベース接続の設定 {#setup-database}

Blitz Authはセッションベースの認証システムを提供するため、セッション情報を保存するためのデータベースが必要です。新しいBlitzアプリにはデフォルトでPrismaセットアップがありますが、このパッケージはデータベースに依存しないため、このガイドではPrismaを使用する場合としない場合の2つのオプションについて説明します。

Prismaを使用したくない場合は、[Prismaなしのセクション](#without-prisma)に進んでください。

#### Prismaを使用する場合 {#with-prisma}

まず、`prisma`と`@prisma/client`をインストールします。

```sh
yarn add prisma @prisma/client
```

次に、CLIを使用して新しいPrismaクライアントを初期化します。

```sh
yarn prisma init --datasource-provider sqlite
```

また、新しいPrismaクライアントを作成する必要があります。そのために、`prisma`ディレクトリに`index.ts`ファイルを作成し、次の内容を追加します。

```ts
// prisma/index.ts
import { PrismaClient } from "@prisma/client"

export * from "@prisma/client"
const db = new PrismaClient()
export default db
```

##### Prismaスキーマの修正

次に、`prisma/schema.prisma`ファイルを更新し、`User`、`Session`、および`Token`データベースモデルを追加します。

```prisma
model User {
  id             Int      @id @default(autoincrement())
  email          String   @unique
  hashedPassword String?
  sessions Session[]
}

model Session {
  id                 Int       @id @default(autoincrement())
  expiresAt          DateTime?
  handle             String    @unique
  hashedSessionToken String?
  antiCSRFToken      String?
  publicData         String?
  privateData        String?
  user   User? @relation(fields: [userId], references: [id])
  userId Int?
}
```

データベースに変更を適用し、PrismaのTypeScriptタイプを生成するには、`yarn prisma migrate dev`を実行します。

次のステップに進みます：[認証ロジックの追加](/docs/blitz-auth-with-next/add-auth-logic)。

#### Prismaなしの場合 {#without-prisma}

Blitz Authの設定における`storage`プロパティは、[`SessionConfigMethods`インターフェースを実装するオブジェクトを受け入れます](/docs/auth-config#customize-session-persistence-and-database-access)。メソッドは以下の通りです。

- `getSession`
- `getSessions`
- `createSession`
- `updateSession`
- `deleteSession`

任意のデータベースまたはAPIを使用できますが、このガイドではRedisの例を示します。

```ts
import IoRedis from "ioredis"
import { setupBlitz } from "@blitzjs/next"
import {
  AuthServerPlugin,
  simpleRolesIsAuthorized,
  SessionModel,
  Session,
} from "@blitzjs/auth"

const dbs: Record<string, IoRedis.Redis | undefined> = {
  default: undefined,
  auth: undefined,
}

export function getRedis(): IoRedis.Redis {
  if (dbs.default) {
    return dbs.default
  }
  return (dbs.default = createRedis(0))
}

export function getAuthRedis(): IoRedis.Redis {
  if (dbs.auth) {


 return dbs.auth
  }
  return (dbs.auth = createRedis(1))
}

export function createRedis(db: number) {
  return new IoRedis({
    port: 6379,
    host: "localhost",
    keepAlive: 60,
    keyPrefix: "auth:",
    db,
  })
}

const { gSSP, gSP, api } = setupBlitz({
  plugins: [
    AuthServerPlugin({
      cookiePrefix: "blitz-app-prefix",
      isAuthorized: simpleRolesIsAuthorized,
      storage: {
        createSession: (session: SessionModel): Promise<SessionModel> => {
          return new Promise<SessionModel>((resolve, reject) => {
            getAuthRedis().set(
              `token:${session.handle}`,
              JSON.stringify(session),
              (err) => {
                if (err) {
                  reject(err)
                } else {
                  getAuthRedis().lpush(
                    `device:${String(session.userId)}`,
                    session.handle
                  )
                  resolve(session)
                }
              }
            )
          })
        },
        deleteSession(handle: string): Promise<SessionModel> {
          return new Promise<SessionModel>((resolve, reject) => {
            getAuthRedis()
              .get(`token:${handle}`)
              .then((result) => {
                if (result) {
                  const session = JSON.parse(result) as SessionModel
                  const userId = session.userId as unknown as string
                  getAuthRedis().lrem(userId, 0, handle).catch(reject)
                }
                getAuthRedis().del(handle, (err) => {
                  if (err) {
                    reject(err)
                  } else {
                    resolve({ handle })
                  }
                })
              })
          })
        },
        getSession(handle: string): Promise<SessionModel | null> {
          return new Promise<SessionModel | null>((resolve, reject) => {
            getAuthRedis()
              .get(`token:${handle}`)
              .then((data: string | null) => {
                if (data) {
                  resolve(JSON.parse(data))
                } else {
                  resolve(null)
                }
              })
              .catch(reject)
          })
        },
        getSessions(
          userId: Session.PublicData["userId"]
        ): Promise<SessionModel[]> {
          return new Promise<SessionModel[]>((resolve, reject) => {
            getAuthRedis()
              .lrange(`device:${String(userId)}`, 0, -1)
              .then((result) => {
                if (result) {
                  resolve(
                    result.map((handle) => {
                      return this.getSession(handle)
                    })
                  )
                } else {
                  resolve([])
                }
              })
              .catch(reject)
          })
        },
        updateSession(
          handle: string,
          session: Partial<SessionModel>
        ): Promise<SessionModel> {
          return new Promise<SessionModel>((resolve, reject) => {
            getAuthRedis()
              .get(`token:${handle}`)
              .then((result) => {
                if (result) {
                  const oldSession = JSON.parse(result) as SessionModel
                  const merge = Object.assign(oldSession, session)
                  getAuthRedis()
                    .set(`token:${handle}`, JSON.stringify(merge))
                    .catch(reject)
                }
                reject(new Error("cant update session"))
              })
          })
        },
      },
    }),
  ],
})
```

## 認証ロジックの追加 {#add-auth-logic}

設定部分が完了したので、Blitz Authを使用して簡単な認証ロジックを実装します。このガイドでは、サインアップ、ログイン、およびログアウトをカバーします。

### `pages/api/signup.ts`ファイルを追加 {#add-signup-route}

最初に、`signup`という新しいAPIルートを作成します。その中で、`src/blitz-server.ts`の`setupBlitzServer`関数から取得した`api`関数を使用します。これは、Blitzの`ctx`オブジェクトへのアクセスを提供するNextのAPIハンドラのラッパーです。このハンドラ内で、ユーザーが提供したパスワードをハッシュ化するために`@blitzjs/auth`の`SecurePassword`を使用します。次に、データベースに新しいユーザーを作成し、`$create`メソッドを使用して新しい認証済みセッションを作成します。`session.$create`に提供するオブジェクトはパブリックデータです。これは、`types.ts`ファイルで指定したのと同じプロパティを含みます。最後に、クライアントに応答を送信します。

注：このガイドをシンプルに保つためにエラーハンドリングは行っていません。アプリケーションで使用する前に、適宜拡張および修正してください。

```ts
// pages/api/signup.ts

import { SecurePassword } from "@blitzjs/auth"
import { api } from "../../src/blitz-server"
import db from "../../prisma"

const signup = api(async (req, res, ctx) => {
  // TODO: ランタイム検証（例えばzodを使用して）を追加してパスワードの長さを確認することができます
  const hashedPassword = await SecurePassword.hash(req.body.password)

  const email = req.body.email
  const user = await db.user.create({
    data: { email, hashedPassword, role: "user" },
    select: { id: true, name: true, email: true, role: true },
  })
  await ctx.session.$create({
    userId: user.id,
    email: user.email,
    role: "USER",
  })
  res
    .status(200)
    .json({ userId: ctx.session.userId, ...user, email: req.query.email })
})

export default signup
```

### サインアップフォームの`onSubmit`メソッドに`/signup`フェッチコールを追加 {#add-signup-fetch-call}

バックエンドのサインアップロジックがあるので、クライアントから新しいエンドポイントを呼び出します。例えば、ユーザーがサインアップフォームを送信したときです。呼び出しは次のようになります。

```tsx
import { getAntiCSRFToken } from "@blitzjs/auth"

// リクエストの処理
const antiCSRFToken = getAntiCSRFToken()
await fetch("/api/signup", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "anti-csrf": antiCSRFToken,
  },
  body: JSON.stringify({ email, password }),
})
```

ここで注目すべきは、`getAntiCSRFToken`の使用です。クライアントからAPIルートへのリクエストを行う際には、`anti-csrf`ヘッダーを含める必要があります。詳細は[こちら](https://blitzjs.com/docs/session-management#manual-api-requests)を参照してください。

### `pages/api/login.ts`を追加 {#add-login-route}

前述のように、新しいAPIルートを追加します。その中で、ログイン資格情報を確認するための`authenticateUser`関数を追加し、正しければ、サインアップハンドラと同様に新しい認証済みセッションを作成します。

```ts
// pages/api/login.ts

import { SecurePassword } from "@blitzjs/auth"
import { AuthenticationError } from "blitz"
import db from "../../prisma"
import { api } from "../../src/blitz-server"

export const authenticateUser = async (
  email: string,
  password: string
) => {
  const user = await db.user.findFirst({ where: { email } })
  if (!user) throw new AuthenticationError()
  const result = await SecurePassword.verify(
    user.hashedPassword,
    password
  )
  if (result === SecurePassword.VALID_NEEDS_REHASH) {
    // より安全なハッシュでパスワードをアップグレード
    const improvedHash = await SecurePassword.hash(password)
    await db.user.update({
      where: { id: user.id },
      data: { hashedPassword: improvedHash },
    })
  }
  const { hashedPassword, ...rest } = user
  return rest
}

const login = api(async (req, res, ctx) => {
  const email = req.body.email
  const password = req.body.password
  const user = await authenticateUser(email, password)
  await ctx.session.$create({
    email: user.email,
    userId: user.id,
    role: "USER",
  })
  res
    .status(200)
    .json({ email: req.query.email, userId: ctx.session.userId })
})

export default login
```

### ログインフォームの`onSubmit`メソッドに`/login`フェッチコールを追加 {#add-login-fetch-call}

クライアント側では、サインアップと同様に`/api/login`にリクエストを送信する必要があります。

```tsx
import { getAntiCSRFToken } from "@blitzjs/auth"

const antiCSRFToken = getAntiCSRFToken()
await fetch("/api/login", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "

anti-csrf": antiCSRFToken,
  },
  body: JSON.stringify({ email, password }),
})
```

### `pages/api/logout.ts`を追加 {#add-logout-route}

API側で最後に行うことはログアウトハンドラを追加することです。内部でセッションを削除し、ユーザーをログアウトする`$revoke`関数を使用します。

```ts
// pages/api/logout.ts
import { api } from "../../src/blitz-server"

const logout = api(async (_req, res, ctx) => {
  await ctx.session.$revoke()
  res.status(200).json({ message: "Logged out" })
})

export default logout
```

## APIルート、`getServerSideProps`、および`getStaticProps`での認証の使用 {#using-auth-gssp-gsp}

Blitz AuthセッションのメソッドをNext.js APIルートで使用する方法はすでに見ました。同様のことを`getServerSideProps`と`getStaticProps`でも行うことができます。`createBlitzServer`は`gSSP`と`gSP`関数を返します。これらは`getServerSideProps`と`getStaticProps`のラッパーです。使用例：

```tsx
// MyPage.tsx
import { gSSP } from "src/blitz-server"

export const getServerSideProps = gSSP(async ({ ctx }) => {
  const session = ctx.session
  // session.$authorize
  // session.$setPublicData
  // など
})
```

## クライアント側でのセッションアクセス {#access-session-on-the-client}

Blitz Authは、`PublicData`と`isLoading`プロパティを返す`useSession()`フックを提供します。このフックはアプリケーションのどこでも使用できます。

注：`useSession()`はデフォルトでサスペンスを使用するため、ツリー内の上位に`<Suspense>`コンポーネントが必要です。または、`useSession({suspense: false})`を設定してサスペンスを無効にすることができます。

使用例：

```tsx
import { useSession } from "@blitzjs/auth"

function Component() {
  const session = useSession()

  const userId = session.userId
  const role = session.role

  return /*... */
}
```

## ページに認証を追加 {#adding-auth-to-pages}

これまでに行ったのはBlitz Auth機能の一部です。ページを保護する方法も簡単に探ってみましょう。Blitz Authでは、ページやレイアウトに`authenticate`または`redirectAuthenticatedTo`プロパティを追加できます。それらを使用するためには、`App`コンポーネントを`withBlitz` HOCでラップする必要があります。まだない場合は、`_app.tsx`ファイルを`pages`ディレクトリに追加します。このファイルでは、`App`コンポーネントを`withBlitz`でラップする必要があります。

```tsx
import { AppProps } from "next/app"
import React from "react"
import { withBlitz } from "../src/blitz-client"

function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
export default withBlitz(App)
```

注：`withBlitz`は現在、新しいNext 13レイアウトでは機能しません。サポートを追加するまで、古い`pages/_app.tsx`を使用する必要があります。

では、ページに戻りましょう。`signup.tsx`と`login.tsx`では、`redirectAuthenticatedTo`を使用します。

```tsx
SignupPage.redirectAuthenticatedTo = "/user"
export default SignupPage
```

そして、`login.tsx`では：

```tsx
LoginPage.redirectAuthenticatedTo = "/user"
export default LoginPage
```

`user.tsx`では、`authenticate`を使用し、認証されていないユーザーが訪問しようとした場合にログインページにリダイレクトします。

```tsx
UserPage.authenticate = { redirectTo: "/login" }
export default UserPage
```

他のBlitz Authの機能を探ることに興味がある場合、フックやユーティリティがたくさんあります。[ドキュメント](https://blitzjs.com/docs/auth-utils)をチェックしてください。

## まとめ {#summary}

このガイドでは、Blitz AuthのPrisma設定、新しいNext.jsアプリへのBlitz Authの追加、および基本的な認証フローの実装をカバーしました。また、Blitz Authを使用してページを保護する方法も探りました。

サインアップ、ログイン、ログアウト、パスワードリセットを備えた完全なBlitzアプリを見たい場合は、[Blitzのリポジトリの例](https://github.com/blitz-js/blitz/tree/main/apps/toolkit-app)をチェックするか、`npx blitz new my-new-blitz-app`を実行して新しいプロダクション準備済みBlitzアプリを生成できます。

このガイドのセットアップを含むリポジトリは[こちら](https://github.com/beerose/next13-blitz-auth)で入手できます。

Blitz.jsについてもっと知りたい場合は、以下のリソースをご覧ください。

- [Blitz.js Documentation](https://blitzjs.com/docs/) — Blitz.jsについて学ぶ。
- [Blitz Auth Documentation](https://blitzjs.com/docs/auth) — Blitz Authプラグインについて学ぶ。

フィードバックがある場合は、[Discord](https://discord.blitzjs.com/)または[GitHub](https://github.com/blitz-js/blitz)でご連絡ください。
