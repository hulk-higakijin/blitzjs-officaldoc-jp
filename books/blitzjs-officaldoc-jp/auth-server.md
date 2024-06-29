```mdx
---
title: 認証サーバーサイドAPI
sidebar_label: サーバーサイドAPI
---

## クエリとミューテーションで {#in-queries-and-mutations}

全てのクエリとミューテーションにおいて、`ctx`の2番目のパラメータとして提供される`SessionContext`を利用できます。

```ts
// src/queries/someQuery.ts
import { Ctx } from "blitz"

export default async function someQuery(input: any, ctx: Ctx) {
  // SessionContextクラスにアクセス
  ctx.session.userId
  ctx.session.role
  ctx.session.$create(/*...*/)

  return
}
```

## `getServerSideProps` {#get-server-side-props}

`getServerSideProps`内部でセッションコンテキストを取得するには、`src/blitz-server`からエクスポートされる`gSSP`関数でラップします。

```ts
import { SessionContext } from "@blitzjs/auth"
import { gSSP } from "src/blitz-server"

type Props = {
  userId: unknown
  publicData: SessionContext["$publicData"]
}

export const getServerSideProps = gSSP<Props>(async ({ ctx }) => {
  const { session } = ctx
  return {
    props: {
      userId: session.userId,
      publicData: session.$publicData,
      publishedAt: new Date(0),
    },
  }
})

function PageWithGssp(props: Props) {
  return <div>{JSON.stringify(props, null, 2)}</div>
}

export default PageWithGssp
```

## APIルート {#api-routes}

APIルート内でセッションコンテキストを取得するには、`src/blitz-server`からエクスポートされる`api`関数でラップします。

```ts
import { api } from "src/blitz-server"
import db from "db"

export default api(async (_req, res, ctx) => {
  ctx.session.$authorize()
  const publicData = ctx.session.$publicData

  res.status(200).json({
    userId: ctx.session.userId,
    publicData: { ...publicData },
  })
})
```

## `generateToken()` {#generate-token}

#### `generateToken(numberOfCharacters: number = 32) => string`

これは、パスワードリセットなどのためのトークン生成に使える[nanoid](https://github.com/ai/nanoid)の便利なラッパーです。

#### 使用例

```ts
import { generateToken } from "@blitzjs/auth"

const token = generateToken()
```

## `hash256()` {#hash256}

#### `hash256(value: string) => string`

これは、`sha256`アルゴリズムを使用して文字列をハッシュするためのノード[crypto](https://nodejs.org/api/crypto.html)モジュールを使用した便利なラッパーです。データベースに保存する前にパスワードリセットトークンをハッシュするために使用されます。

Hash256はAPIキーのような文字列をデータベースに保存する場合にも便利です。返されるハッシュは与えられた文字列に対して常に同じなので、参照する唯一の値がハッシュ化されたキーであっても、APIキーがデータベースに存在することを確認できます。

#### 使用例

```ts
import { hash256 } from "@blitzjs/auth"

const hashedToken = hash256(token)
```

## `SecurePassword` {#secure-password}

`SecurePassword`は、パスワードのハッシュ化とパスワードハッシュの検証をサポートする[secure-password](https://github.com/emilbayes/secure-password)の便利なラッパーです。

```ts
import { SecurePassword } from "@blitzjs/auth"

await SecurePassword.hash(password)
await SecurePassword.verify(passwordHash, password)
```

#### `SecurePassword.hash(password: string) => Promise<string>`

これは、ユーザーが新しいパスワードを設定する際に使用します。

パスワード文字列を受け取り、安全なハッシュを生成してデータベースに保存します。

`SecurePassword.hash`は同じ文字列に対して異なるハッシュを返すため、ハッシュを比較するために`SecurePassword.verify`が必要です。

#### `SecurePassword.verify(passwordHash: string, password: string) => Promise<ResultCode>`

これは、ユーザーがログインする際に正しいパスワードを使用したかどうかを検証するために使用します。

データベースのパスワードハッシュと与えられたパスワードを受け取ります。与えられたパスワードが正しい場合に結果コードを返し、間違っている場合は`AuthenticationError`をスローします。

##### 結果コード

**`SecurePassword.VALID`**

パスワードが検証され、有効です。

**`SecurePassword.VALID_NEEDS_REHASH`**

パスワードが検証され、有効ですが、新しいパラメータで再ハッシュする必要があります。

**`SecurePassword.HASH_BYTES`**

`hash`と`hashSync`によって返され、`verify`と`verifySync`によって使用される`hash`バッファーのサイズ。

#### 使用例

```ts
import { AuthenticationError } from "blitz"
import { SecurePassword } from "@blitzjs/auth"
import db from "db"

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
    // より安全なハッシュでパスワードハッシュをアップグレード
    const improvedHash = await SecurePassword.hash(password)
    await db.user.update({
      where: { id: user.id },
      data: { hashedPassword: improvedHash },
    })
  }

  const { hashedPassword, ...rest } = user
  return rest
}
```

## `setPublicDataForUser()` {#set-public-data-for-user}

#### `setPublicDataForUser(userId: PublicData['userId'], publicData: Record<any, any>) => void`

これは、ユーザーのセッションの`publicData`を更新するために使用できます。ユーザーのロールを変更する際に役立ちます。新しい権限は、ユーザーが次回リクエストを行うときにすぐに適用されます。

#### 使用例

```ts
import { setPublicDataForUser } from "@blitzjs/auth"
import db from "db"

export const updateUserRole = async (
  userId: PublicData["userId"],
  role: string
) => {
  // ユーザのロールを更新
  await db.user.update({ where: { id: userId }, data: { role } })
  // すべてのアクティブなセッションでロールを更新
  await setPublicDataForUser(userId, { role })
}
```
```