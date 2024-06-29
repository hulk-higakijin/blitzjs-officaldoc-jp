```mdx
---
title: Next.jsでのBlitz認証
sidebar_label: Next.jsでのBlitz認証
---

このガイドでは、Next.jsアプリにBlitz認証を追加する方法を説明します。Next.jsとBlitzのセットアップ、基本的な認証ロジックの作成、NextのページおよびAPIハンドラー内でのBlitz認証機能の使用について説明します。

ガイドを進める際に行き詰まった場合は、[このリポジトリ](https://github.com/beerose/next13-blitz-auth)を参照してください。

## 新しいNext.jsアプリの作成 {#create-new-next-app}

まず、`create-next-app`を使って新しいNextアプリケーションを作成します:

```sh
npx create-next-app@latest
```

その後、`yarn dev`を実行し、ブラウザで`http://localhost:3000`を開いて確認できます。

## Blitzのセットアップ {#blitz-setup}

### 依存関係のインストール {#install-dependencies}

次のステップとして、Blitz.jsのパッケージをインストールする必要があります:

- `blitz` — Blitzのコアパッケージで、Blitz CLI、ユーティリティ、他のBlitzパッケージで使用される基本機能が含まれています。
- `@blitzjs/next` — Next.jsフレームワークアダプターで、NextプロジェクトでBlitzを初期化するために必要です。
- `@blitzjs/auth` — このガイドで紹介する認証プラグインです。

```sh
yarn add blitz @blitzjs/next @blitzjs/auth
```

アプリでBlitzプラグインを機能させるには、クライアント側とサーバー側の2つの設定ファイルが必要です。まず、`src`ディレクトリを作成し、クライアント側の設定ファイルから始めましょう。

### `blitz-client.ts`ファイルを追加 {#add-blitz-client}

新しいファイル`blitz-client.ts`を作成し、以下の内容を追加します:

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

ここでは、`@blitzjs/next`の`setupBlitzClient`を使用して、使用したいプラグインの設定を提供しています。この場合、認証プラグインにのみ関心があります。設定する必要があるのはクッキープリフィックスだけで、残りはBlitzのデフォルトに依存します。`authConfig`を個別の変数として持つ理由は、サーバー設定で再利用できるようにするためです。

### `blitz-server.ts`ファイルを追加 {#add-blitz-server}

次に、サーバー側の設定に進みます。同じディレクトリに`blitz-server.ts`ファイルを追加します:

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

このファイルでは、より多くのことを行っています:

- `@blitzjs/next`から`setupBlitzServer`を使用しています（クライアント設定ファイルと同様に）。
- 認証プラグインのサーバー側設定では、2つの追加項目が必要です:
  - `storage` : Blitzはセッション情報をデータベースに保存するため、Blitz認証がストレージにアクセスできる方法を指定する必要があります。
  - `isAuthorized`: ユーザーロールがページやその他の保護されたコードにアクセスできるかどうかを判断する関数です。ここではBlitz認証で提供される`simpleRolesIsAuthorized`を使用します。詳細はhttps://blitzjs.com/docs/authorization#is-authorized-adaptersで確認できます。

### `types.ts`ファイルを追加 {#add-types}

`blitz-server.ts`ファイルを作成した後、Blitz認証設定の`storage`プロパティに関するいくつかのTypeScriptの問題に気付いたかもしれません。これは、Blitzが`Session`オブジェクトの見た目を設定するための型拡張を使用しているためです。プロジェクトのルートに`types.ts`ファイルを作成しましょう:

```
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

Blitz認証はセッションベースの認証システムを提供するため、セッション情報を格納するデータベースが必要です。新しいBlitzアプリにはデフォルトでPrismaの設定が含まれていますが、このパッケージはデータベースに依存しないため、このガイドではPrismaを使う場合と使わない場合の2つのオプションについて説明します。

Prismaを使用しない場合は[Without Prismaセクション](#without-prisma)に進んでください。

#### Prismaを使用する場合 {#with-prisma}

まず、`prisma`と`@prisma/client`をインストールします:

```sh
yarn add prisma @prisma/client
```

次に、そのCLIを使用して新しいPrismaクライアントを初期化します:

```sh
    yarn prisma init --datasource-provider sqlite
```

新しいPrismaクライアントも作成する必要があります。そのため、`prisma`ディレクトリに`index.ts`ファイルを作成し、以下の内容を追加します:

```ts
// prisma/index.ts
import { PrismaClient } from "@prisma/client"

export * from "@prisma/client"
const db = new PrismaClient()
export default db
```

##### Prismaスキーマの修正

次に、`prisma/schema.prisma`ファイルを更新して、`User`、`Session`、および`Token`データベースモデルを追加します:

```
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

変更をデータベースに適用し、PrismaのTypeScript型を生成するには、`yarn prisma migrate dev`を実行します。

次のステップに進みます:
[認証ロジックの追加](/docs/blitz-auth-with-next/add-auth-logic).

#### Prismaを使用しない場合 {#without-prisma}

Blitz認証設定の`storage`プロパティは、[`SessionConfigMethods`インターフェースを実装するオブジェクトを受け入れます](/docs/auth-config#customize-session-persistence-and-database-access)。
これらのメソッドは次の通りです:

- `getSession`
- `getSessions`
- `createSession`
- `updateSession`
- `deleteSession`

どのデータベースやAPIでも使用できますが、このガイドではRedisの例を示します:

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

セットアップ部分が完了したので、Blitz認証を使用してシンプルな認証ロジックを実装することができます。このガイドでは、サインアップ、ログイン、ログアウトをカバーします。

### `pages/api/signup.ts`ファイルを追加 {#add-signup-route}

まず、`signup`という新しいAPIルートを作成します。その中で、`src/blitz-server.ts`の`setupBlitzServer`関数から取得した`api`関数を使用します。これは、Blitzの`ctx`オブジェクトへのアクセスを提供するNextのAPIハンドラーのラッパーです。このオブジェクトには認証関連のメソッドとプロパティが含まれています。

ハンドラーの中では、ユーザーが提供したパスワードをハッシュ化するために`@blitzjs/auth`から`SecurePassword`を使用します。次に、データベースに新しいユーザーを作成し、`$create`メソッドを使用して新しい認証済みセッションを作成します。`session.$create`に提供するオブジェクトはPublicDataです。これは`types.ts`ファイルで指定したプロパティと同じプロパティを含んでいます。最後に、クライアントに応答を送信します。

注意: このガイドでは、エラーハンドリングは行っていません。これは、ガイドを最小限に保ち、Next.jsでBlitz認証を設定する方法に焦点を当てるためです。アプリケーションで使用する前に、適宜拡張および修正する必要があります。

```ts
// pages/api/signup.ts

import { SecurePassword } from "@blitzjs/auth"
import { api } from "../../src/blitz-server"
import db from "../../prisma"

const signup = api(async (req, res, ctx) => {
  // TODO: パスワードの長さを確保するためのランタイムバリデーション（例：zod）を追加することができます。
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

### サインアップフォームの`onSubmit`メソッドに`/signup`のフェッチコールを追加 {#add-signup-fetch-call}

サインアップのバックエンドロジックがあるので、クライアントから新しいエンドポイントを呼び出すことができます。例えば、ユーザーがサインアップフォームを送信したときに呼び出します。この呼び出しは以下のようになります:

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

ここで注目すべき点は、`getAntiCSRFToken`の使用です。クライアントからAPIルートにリクエストを送信する場合、`anti-csrf`ヘッダーを含める必要があります。詳細は[こちら](https://blitzjs.com/docs/session-management#manual-api-requests)で確認できます。

### `pages/api/login.ts`ファイルを追加 {#add-login-route}

前と同様に、新しいAPIルートを追加します。その中で、ログイン資格情報を確認するための`authenticateUser`関数を追加し、正しければ、サインアップハンドラーで行ったように、新しい認証済みセッションを作成します。

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
    // より安全なハッシュでハッシュ化されたパスワードをアップグレード
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

### ログインフォームの`onSubmit`メソッドに`/login`のフェッチコール