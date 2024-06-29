```mdx
---
title: 認証とセキュリティ
sidebar_label: 認証とセキュリティ
---

認証は、アプリケーション内のデータやページへのアクセスを許可または禁止する行為です。

## データを保護する {#secure-your-data}

データを保護するには、保護したいすべてのクエリおよびミューテーションの内部で
[`ctx.session.$authorize()`](#ctxsessionauthorize) を呼び出してください。 
（あるいは [`resolver.pipe`](./resolver-server-utilities#resolverpipe) を使用している場合は、 
[`resolver.authorize`](./resolver-server-utilities#resolverauthorize) を使用してください）。
同様に [API ルート](https://nextjs.org/docs/api-routes/introduction) を保護することもできます。

これらはユーザーがログインしていない場合には `AuthenticationError` がスローされ、必要な権限がない場合には `AuthorizationError` がスローされます。

```ts
import { resolver } from "@blitzjs/rpc"
import db from "db"
import * as z from "zod"

const CreateProject = z
  .object({
    name: z.string(),
  })
  .nonstrict()

export default resolver.pipe(
  resolver.zod(CreateProject),
  // ハイライト開始
  resolver.authorize(),
  // ハイライト終了
  async (input, ctx) => {
    // *: マルチテナントアプリの場合、正しいテナントを確認するための検証を追加する必要があります
    const projects = await db.projects.create({ data: input })
    return projects
  }
)
```

あるいは

```ts
import {Ctx} from "blitz"
import db from "db"
import * as z from "zod"

const CreateProject = z
  .object({
    name: z.string(),
  }).nonstrict()
type CreateProjectType = z.infer<typeof CreateProject>

export default function createProject(input: CreateProjectType, ctx: Ctx) {
  const data = CreateProject.parse(input)

  // ハイライト開始
  ctx.session.$authorize()
  // ハイライト終了

  // *: マルチテナントアプリの場合、正しいテナントを確認するための検証を追加する必要があります
  const projects = await db.projects.create({data})
  return projects
}
```

### 入力検証 {#input-validation}

セキュリティのために、ミューテーション内のすべての入力値を検証することが非常に重要です。
[zod](https://github.com/colinhacks/zod) を使うことを推奨しており、これはすべてのコード生成テンプレートに含まれています。
これを行わないと、ユーザーが禁止されているデータにアクセスしたりアクションを実行したりできる可能性があります。

```ts
import { resolver } from "@blitzjs/rpc"
import db from "db"
import * as z from "zod"

// ハイライト開始
const CreateProject = z
  .object({
    name: z.string(),
  })
  .nonstrict()
// ハイライト終了

export default resolver.pipe(
  // ハイライト開始
  resolver.zod(CreateProject),
  // ハイライト終了
  resolver.authorize(),
  async (input, ctx) => {
    // *: マルチテナントアプリの場合、正しいテナントを確認するための検証を追加する必要があります
    const projects = await db.projects.create({ data: input })
    return projects
  }
)
```

## ページを保護する {#secure-your-pages}

ユーザーがログインする必要があるすべてのページに `Page.authenticate = true` を設定してください。
ユーザーがログインしていない場合、`AuthenticationError` がスローされ、トップレベルのエラーバウンダリでキャッチされます。

ユーザーの役割に基づいてページを認証する場合：

```tsx
// ユーザーはログインしている必要があり、"ADMIN" または "USER" の役割を持っている必要があります
Page.authenticate = { role: "ADMIN" }
// または役割をリストとして設定し、ユーザーは"ADMIN" または "USER" の役割を持っている必要があります
Page.authenticate = { role: ["ADMIN", "USER"] }
```

あるいは、ユーザーをリダイレクトしたい場合は、
`Page.authenticate = {redirectTo: '/login'}` を設定します：

```tsx
const Page: BlitzPage = () => {
  return <div>{/* ... */}</div>
}

// ハイライト開始
Page.authenticate = true
// または Page.authenticate = {redirectTo: '/login'}
// または Page.authenticate = {redirectTo: Routes.Login()}
// または Page.authenticate = { role: "ADMIN", redirectTo: "/login" }
// ユーザーは "ADMIN" の役割を持っている必要があり、そうでない場合はリダイレクトされます
// ハイライト終了

export default Page
```

### ログインしているユーザーをリダイレクトする {#redirecting-logged-in-users}

ログアウトしたユーザー専用のページ（ログインページやサインアップページなど）には、
`Page.redirectAuthenticatedTo = '/'` を設定して、ログインしているユーザーを別のページに自動的にリダイレクトします。

```tsx
import { Routes } from "@blitzjs/next"

const Page: BlitzPage = () => {
  return <div>{/* ... */}</div>
}

// ハイライト開始
// フルパスを使用
Page.redirectAuthenticatedTo = "/"
// ルートマニフェストを使用
Page.redirectAuthenticatedTo = Routes.Home()
// 関数を使用
Page.redirectAuthenticatedTo = ({ session }) =>
  session.role === "admin" ? "/admin" : Routes.Home()
// ハイライト終了

export default Page
```

### セキュアなレイアウト {#secure-layouts}

レイアウトをページと同様に保護することができます：

```tsx
const Layout = ({ children }) => {
  return <div>{children}</div>
}

// ハイライト開始
Layout.authenticate = true
// または Layout.authenticate = {redirectTo: '/login'}
// または Layout.authenticate = {redirectTo: Routes.Login()}
// または Layout.redirectAuthenticatedTo = Routes.Home()
// ハイライト終了

const Page = () => {
  return <div>{/* ... */}</div>
}

Page.getLayout = (page) => <Layout>{page}</Layout>

export default Page
```

Blitzは`getLayout`の応答から最初に見つけたコンポーネントを取り、それが`authenticate`キーまたは`redirectAuthenticatedTo`キーを持つ場合、その値を認証に使用します。

これらの値は、個々のページごとに`Page.authenticate`または`Page.redirectAuthenticatedTo`で上書きできます：

```tsx
const Layout: BlitzLayout = ({ children }) => {
  return <div>{children}</div>
}

Layout.authenticate = true

const Page: BlitzPage = () => {
  return <div>{/* ... */}</div>
}

Page.getLayout = (page) => <Layout>{page}</Layout>
// ハイライト開始
Page.authenticate = false
// ハイライト終了

export default Page
```

## 認可されていないユーザーのためのUX {#ux-for-unauthorized-users}

`/login` ページへのリダイレクトを使用することはできますが、リダイレクトの代わりにエラーバウンダリを使用することを推奨します。

Reactでは、UIのエラーをキャッチする方法は[エラーバウンダリ](https://reactjs.org/docs/error-boundaries.html) を利用することです。

これらのエラーがアプリのどこからでも処理されるように、`_app.tsx` 内にトップレベルのエラーバウンダリが必要です。必要であれば、アプリの他の場所にもエラーバウンダリを配置できます。

新しいBlitzアプリのデフォルトのエラーハンドリングは以下の通りです：

- `AuthenticationError` がスローされた場合、別のルートにリダイレクトする代わりにユーザーに直接ログインフォームを表示します。これによりリダイレクトURLの管理が不要になります。一度ログインすると、エラーバウンダリがリセットされ、ユーザーは元のページにアクセスできます。
- `AuthorizationError` がスローされた場合、その旨を表すエラーを表示します。

以下は `src/pages/_app.tsx` にあるデフォルトの `RootErrorFallback` です。必要に応じてカスタマイズできます。

```tsx
import { AuthenticationError, AuthorizationError } from "blitz"
import { ErrorComponent, ErrorFallbackProps } from "@blitzjs/next"

function RootErrorFallback({
  error,
  resetErrorBoundary,
}: ErrorFallbackProps) {
  if (error instanceof AuthenticationError) {
    return <LoginForm onSuccess={resetErrorBoundary} />
  } else if (error instanceof AuthorizationError) {
    return (
      <ErrorComponent
        statusCode={error.statusCode}
        title="申し訳ありませんが、このページにアクセスする権限がありません"
      />
    )
  } else {
    return (
      <ErrorComponent
        statusCode={error.statusCode || 400}
        title={error.message || error.name}
      />
    )
  }
}
```

Blitzでのエラーハンドリングの詳細については、[Error Handling documentation](./error-handling) を参照してください。

## ユーザーの役割に基づいた異なるコンテンツの表示 {#displaying-different-content-based-on-user-role}

UI内でユーザーの役割をチェックする方法は2つあります。

#### `useSession()`

最初の方法は、[`useSession()`](./session-management#session-on-the-client) フックを使用してセッションの `publicData` からユーザーの役割を読み取ることです。

これはバックエンドへのネットワーク呼び出しなしでクライアント上で使用可能なので、以下で説明する `useCurrentUser()` アプローチよりも速く使用可能です。

注意：静的なプリレンダリングの性質上、初回レンダリング時には `session` が存在しません。このため、初回ロード時に短い"フラッシュ"が発生します。これを修正するには、`Page.suppressFirstRenderFlicker = true` を設定します。

```tsx
import { useSession } from "@blitzjs/auth"

const session = useSession()

if (session.role === "admin") {
  return /* 管理内容 */
} else {
  return /* 通常内容 */
}
```

#### `useCurrentUser()`

第二の方法は `useCurrentUser()` フックを使用することです。新しいBlitzアプリにはデフォルトで `useCurrentUser()` フックと対応する `getCurrentUser` クエリが含まれています。上記の `useSession()` アプローチとは異なり、`useCurrentUser()` はネットワーク呼び出しが必要なので、遅くなります。ただし、セッションの `publicData` に保存するのが不適切なユーザーデータにアクセスする必要がある場合は、`useSession()` の代わりに `useCurrentUser()` を使用する必要があります。

```tsx
import { useCurrentUser } from "src/users/hooks/useCurrentUser"

const user = useCurrentUser()

if (user.isFunny) {
  return /* 面白い内容 */
} else {
  return /* 通常内容 */
}
```

## `isAuthorized` アダプター {#is-authorized-adapters}

`ctx.session.$isAuthorized()` と `ctx.session.$authorize()` の実装は、`sessionMiddleware()` の設定でアダプターによって定義されます。

##### `ctx.session.$isAuthorized()`

ユーザーが認証されているかどうかを示すブール値を常に返します

##### `ctx.session.$authorize()`

ユーザーが認証されていない場合は **エラーをスロー** します。 これは、クエリやミューテーションを保護するために最も一般的に使用されます。

```ts
import { Ctx } from "blitz"
import { GetUserInput } from "./somewhere"

export default async function getUser({ where }: GetUserInput, ctx: Ctx) {
  // ハイライト開始
  ctx.session.$authorize("admin")
  // ハイライト終了

  return await db.user.findOne({ where })
}
```

#### `simpleRolesIsAuthorized`（新しいアプリのデフォルト）

##### 設定

使用するには、`sessionMiddleware` 設定に追加します（これは新しいアプリではデフォルトで設定されています）。

```js
// blitz.config.js
const { sessionMiddleware, simpleRolesIsAuthorized } = require("blitz")

module.exports = {
  middleware: [
    sessionMiddleware({
      // ハイライト開始
      isAuthorized: simpleRolesIsAuthorized,
      // ハイライト終了
    }),
  ],
}
```

そして、TypeScriptを使用している場合は、`types.ts` に次のように型を設定します：

```ts
import { SimpleRolesIsAuthorized } from "@blitzjs/auth"

type Role = "ADMIN" | "USER"

declare module "@blitzjs/auth" {
  // ハイライト開始
  export interface Session {
    isAuthorized: SimpleRolesIsAuthorized<Role>
  }
  // ハイライト終了
}
```

##### `ctx.session.$isAuthorized(roleOrRoles?: string | string[])`

使用例：

```ts
// ユーザーがログインしていない
ctx.session.$isAuthorized() // false

// 'customer' 役割を持ったユーザーがログインしている
ctx.session.$isAuthorized() // true
ctx.session.$isAuthorized("customer") // true
ctx.session.$isAuthorized("admin") // false
```

##### `ctx.session.$authorize(roleOrRoles?: string | string[])`

使用例：

```ts
// ユーザーがログインしていない
ctx.session.$authorize() // AuthenticationError をスロー

// 'customer' 役割を持ったユーザーがログインしている
ctx.session.$authorize() // 成功 - エラーなし
ctx.session.$authorize("customer") // 成功 - エラーなし
ctx.session.$authorize("admin") // AuthorizationError をスロー
ctx.session.$authorize(["admin", "customer"]) // 成功 - エラーなし
```

### カスタムアダプターの作成 {#making-a-custom-adapter}

`isAuthorized` アダプターは、次の関数シグネチャに準拠する必要があります。

```ts
type CustomIsAuthorizedArgs = {
  ctx: any
  args: [/* session.$authorize(...args) の引数 */]
}
export function customIsAuthorized({
  ctx,
  args,
}: CustomIsAuthorizedArgs) {
  // ctx.session, ctx.session.userId などにアクセスできます
}
```

##### 使用例

ここに、Blitzコアに含まれている `simpleRolesIsAuthorized` アダプターのソースコードがあります（2021年1月26日現在）。

```ts
type SimpleRolesIsAuthorizedArgs = {
  ctx: any
  args: [roleOrRoles?: string | string[]]
}

export function simpleRolesIsAuthorized({
  ctx,
  args,
}: SimpleRolesIsAuthorizedArgs) {
  const [roleOrRoles, options = {}] = args
  const condition = options.if ?? true

  // 役割が要求されていないため、すべての役割が許可される
  if (!roleOrRoles) return true
  // 条件がfalseの場合、役割を強制しない
  if (!condition) return true

  const rolesToAuthorize = []
  if (Array.isArray(roleOrRoles)) {
    rolesToAuthorize.push(...roleOrRoles)
  } else if (roleOrRoles) {
    rolesToAuthorize.push(roleOrRoles)
  }
  for (const role of rolesToAuthorize) {
    if ((ctx.session as SessionContext).$publicData.roles!.includes(role))
      return true
  }
  return false
}
```
```