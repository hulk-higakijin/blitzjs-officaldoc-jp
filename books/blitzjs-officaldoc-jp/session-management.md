    ```mdx
---
title: セッション
sidebar_label: セッション
---

Blitzには、任意のタイプの認証またはアイデンティティプロバイダーで使用できる組み込みのセッション管理機能があります。

セッション管理は次の機能を果たします：

1. ユーザーがログインしているかどうかのトラッキング
2. ユーザーがログアウトしていても同じユーザーへの複数のリクエストを属性付け
3. CSRF攻撃からの保護

<Card type="note">
  ログイン、ログアウト、およびその他のセッションの変更は、[`SessionContext`](#sessioncontext) オブジェクトを介して行います。このオブジェクトは[サーバーのどこからでもアクセスできます](#access-session-on-the-server)。
</Card>

## 基本 {#session-basics}

### ログイン {#login}

ログインのためには、下記のようなログインミューテーションを提出するフォームコンポーネントがUIにあります。

```ts
// src/auth/mutations/login.ts
import { Ctx } from "blitz"

export default async function login(input: SomeTSInputType, ctx: Ctx) {
  // 1. 入力データを検証
  // 2. ユーザー資格情報を検証
  // 3. ユーザーデータの取得

  // 4. 新しいセッションを作成（ログイン）
  await ctx.session.$create({ userId: user.id, role: user.role }) // highlight-line
}
```

### ログアウト {#logout}

ログアウトのために、UIに下記のようなログアウトミューテーションを提出するボタンがあります。

セッションを取り消すと、すぐにクライアントサイドのクエリキャッシュが削除され、ページ上のすべてのクエリが再フェッチされます。これにより、キャッシュ内の敏感なデータが削除されます。

```ts
// src/auth/mutations/logout.ts
import { Ctx } from "blitz"

export default async function logout(_: any, ctx: Ctx) {
  // 1. 現在のユーザーセッションを取り消し、ログアウトします。
  return await ctx.session.$revoke() // highlight-line
}
```

### 現在のユーザーのセッション公開データを変更 {#change-session-public-data-of-current-user}

各セッションには[`PublicData`](#publicdata)があり、これはクライアントで利用可能であり、任意のJavascriptコードによって読み取ることができるクッキーに保存されているため、第三者によって読み取られる可能性があります。通常、これは現在のユーザーID、ユーザーロール、および場合によっては現在の組織IDを保存するために使用されます。

クエリやミューテーションで次のようにセッションの公開データを変更できます：

```ts
// src/mutations/someMutation.ts
import { Ctx } from "blitz"

export default async function someMutation(input: any, ctx: Ctx) {
  // これにより、入力データが現在のpublicDataにあるものとマージされます
  await ctx.session.$setPublicData({ orgId: 1 }) // highlight-line
}
```

### セッションの有効期限 / 自動更新 {#session-expiration-and-auto-refresh}

セッションは有効期限の1/4が経過すると自動的に更新されます。デフォルトの有効期限は30日です。セッションの[`PublicData`](#publicdata)を変更すると、セッションも自動的に更新されます。セッションの更新後にトークン（アクセストークンおよびanti-CSRF）を更新する必要があります（例：モバイルアプリの場合）。

## 匿名セッション {#anonymous-sessions}

ユーザーがログインしていない場合、匿名セッションが自動的に作成されます。`ctx.session.$setPublicData()`および`ctx.session.$setPrivateData()`をログインしていないユーザーと同様に匿名セッションで使用できます。匿名セッションに設定したデータは、ユーザーがログインすると自動的に認証セッションに転送されます。

匿名セッションはJWTトークンであり、クライアントにhttpOnlyクッキーとして保存され、期限切れにはなりません。

匿名セッションの`PublicData`はセッションJWTに保存され、データベースには保存されません。`session.$setPrivateData()`を呼び出すと匿名セッションがデータベースに保存されます。

最初のネットワークリクエストが行われた時点で、匿名セッションが作成されます。これは、`sessionMiddleware`がそのリクエストのミドルウェアチェーンにある限り、SSRまたはAPI経由で発生します。

このユースケースの一例は、匿名ユーザーのショッピングカートアイテムの保存です。匿名ユーザーが後でサインアップまたはログインすると、その匿名セッションデータは新しい認証済みセッションにマージされます。

匿名セッションの`PublicData`は次のようになります：

```
{
  userId: null,
}
```

## TypeScriptでセッション公開データをカスタマイズ {#customize-session-public-data-in-typescript}

TypeScriptを使用する場合、まず`types.ts`で`Session.PublicData`タイプを次のように更新します：

```diff
import {SessionContext, SimpleRolesIsAuthorized} from "@blitzjs/auth"
import {User} from "db"

// 注：Postgresに切り替えて、ロールタイプのためのDB列挙を使用する必要があります
export type Role = "ADMIN" | "USER"

declare module "@blitzjs/auth" {
  export interface Session {
    isAuthorized: SimpleRolesIsAuthorized<Role>
    PublicData: {
      userId: User["id"]
      role: Role
+     orgId: number
    }
  }
}
```

その後、すべての`ctx.session.$create()`の使用を新しいフィールドを渡すように変更します。

```ts
ctx.session.$create({ userId: 1, role: "ADMIN", orgId: 1 })
```

また、既にログインしているユーザーのセッションデータを更新するために`ctx.session.$setPublicData()`を使用することもできます。これにより、既存の公開データと値がマージされます。

```ts
ctx.session.$setPublicData({ orgId: 1 })
```

クライアント上で公開データにアクセスするには：

```ts
import { useSession } from "@blitzjs/auth"

function SomeComponent() {
  const session = useSession()

  session.orgId

  return /*... */
}
```

サーバー上で公開データにアクセスするには：

```ts
// app/queries/someQuery.ts
import { Ctx } from "blitz"

export default async function someQuery(input: any, ctx: Ctx) {
  // SessionContextクラスにアクセス
  ctx.session.orgId

  return
}
```

## 手動APIリクエスト {#manual-api-requests}

クライアントからAPIルートにリクエストを行う場合、`anti-CSRF`ヘッダーにanti-CSRFトークンを含める必要があります。次のようにします：

```ts
import { getAntiCSRFToken } from "@blitzjs/auth"

const antiCSRFToken = getAntiCSRFToken()

if (antiCSRFToken) {
  // fetchリクエストヘッダー["anti-csrf"] = antiCSRFToken を設定
}
```

そして、APIルートで次のようにsessionContextを取得できます：

```ts
import { getSession } from "@blitzjs/auth"

export default async function ({ req, res }) {
  const session = await getSession(req, res)
  console.log("ユーザーID:", session.userId)

  res.json({ userId })
}
```

## 仕組みの技術的詳細 {#technical-details-of-how-it-works}

認証されたセッションは、データベースに保存される不透明なトークンを使用します。

### 実装の詳細 {#implementation-details}

#### セッションの作成

- ログイン時にサーバーは2つの不透明なトークンを作成します：
  - アクセストークン
  - anti-CSRFトークン
- 両方とも32文字の長さの`string`です。
- アクセストークンは、`httpOnly`、`secure`クッキーを介してフロントエンドに送信されます。
- anti-CSRFトークンは、通常の、`secure`クッキーを介してフロントエンドに送信され、Javascriptから読み取れます。
- アクセストークンのSHA256ハッシュはデータベースに保存されます。このトークンには次のプロパティがマッピングされています：
  - userId
  - 有効期限
  - セッションデータ
- anti-CSRFトークンはアクセストークンと一緒に保存されます。
- 別のセッションが存在する間に新しいセッションを作成すると、ヘッダー/クッキーが変更されます。ただし、以前のセッションは依然として有効です。
- 本格的なプロダクションアプリのために、定期的に期限切れのトークンを削除するcronjobが必要です。

#### セッションの検証

- CSRF保護が必要なリクエストごとに、フロントエンドはリクエストヘッダーにanti-CSRFトークンを送信します。
- 受信したアクセストークンは、データベースにあり、期限が切れていないことを確認することで検証されます。検証ごとにアクセストークンの有効期限が更新されます。
- CSRF攻撃は、受信したanti-CSRFトークン（ヘッダーから）がセッションに関連付けられていることを確認することで防止されます。

#### セッションの取り消し/ログアウト

- これは、データベースからセッションを削除することで行われます。
- ログアウトはさらにクッキーをクリアし、フロントエンドにlocalstorageからanti-CSRFトークンを削除するように指示するヘッダーが送信されます

## タイプ {#types}

#### `SessionContext`

```ts
interface SessionContext extends PublicData {
  /**
   * 匿名の場合はnull
   */
  userId: unknown
  $handle: string | null
  $publicData: PublicData
  $authorize(
    ...args: IsAuthorizedArgs
  ): asserts this is AuthenticatedSessionContext
  $isAuthorized: (
    ...args: IsAuthorizedArgs
  ) => this is AuthenticatedSessionContext
  $create: (
    publicData: PublicData,
    privateData?: Record<any, any>
  ) => Promise<void>
  $revoke: () => Promise<void>
  $revokeAll: () => Promise<void>
  $getPrivateData: () => Promise<Record<any, any>>
  $setPrivateData: (data: Record<any, any>) => Promise<void>
  $setPublicData: (
    data: Partial<Omit<PublicData, "userId">>
  ) => Promise<void>
}
```
    ```