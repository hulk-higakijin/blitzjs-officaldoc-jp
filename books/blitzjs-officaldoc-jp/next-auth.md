---
title: NextAuthでのサードパーティーログイン
---

Blitzは、任意のNextjsアプリケーションでBlitzのセッション管理を使用して、任意の
[NextAuth Provider](https://next-auth.js.org/providers/) を使用できるアダプターを提供します。Blitzのセッション管理は、NextAuthよりも多くの柔軟性と制御を提供します。

## セットアップ {#setup}

### 1. `next.config.js`へのNextAuthアダプターの追加 {#add-next-auth-adapter-next-config}

```ts
const { withNextAuthAdapter } = require("@blitzjs/auth/next-auth")
const { withBlitz } = require("@blitzjs/next")

/**
 * @type {import('@blitzjs/next').BlitzConfig}
 **/
const config = {
  ...
}

module.exports = withBlitz(withNextAuthAdapter(config))
```

### 2. NextAuth APIルートの追加 {#add-the-nextauth-js-api-route}

以下の内容で、新しいAPIルートを`src/pages/api/auth/[...nextauth].ts`に追加します。

```ts
// src/pages/api/auth/[...nextauth].ts
import { api } from "src/blitz-server"
import GithubProvider from "next-auth/providers/github"
import GoogleProvider from "next-auth/providers/google"
import { NextAuthAdapter } from "@blitzjs/auth/next-auth"
import db, { User } from "db"
import { Role } from "types"

// `profile`を以下で正しく型付けするために別途定義する必要があります
const providers = [
  GithubProvider({
    clientId: process.env.GITHUB_CLIENT_ID as string,
    clientSecret: process.env.GITHUB_CLIENT_SECRET as string,
  }),
  GoogleProvider({
    clientId: process.env.GOOGLE_CLIENT_ID as string,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
  }),
]

export default api(
  NextAuthAdapter({
    successRedirectUrl: "/",
    errorRedirectUrl: "/error",
    providers,
    callback: async (user, account, profile, session) => {
      ...
    },
  })
)
```

必要に応じて、APIルートを別のパスに配置できますが、ファイル名は`[...nextauth].js`または`[...nextauth].ts`でなければなりません。

### 設定構造 {#config}

```ts
export type BlitzNextAuthOptions = AuthOptions & {
  // 認証成功後のリダイレクト
  successRedirectUrl: string
  // 意図的なまたはその他の認証エラー後のリダイレクト
  errorRedirectUrl: string
  secureProxy?: boolean
  callback: (
    user: User,
    account: Account,
    // 宣言されたプロバイダーから自動的に推論されます
    profile: Profile,
    session: SessionContext,
    provider: ProviderName
  ) => Promise<void | { redirectUrl: string }>
}
```

#### URLs {#urls}

`NextAuth`アダプターは、インストールされた各戦略ごとに2つのAPIエンドポイントを追加します。
`src/pages/api/auth/[...nextauth].ts`のハンドラーで、次のものが追加されます：

1. `/api/auth/[providerName]/login` - ログインを開始するURL
2. `/api/auth/[providerName]/callback` - ログインを完了するためのコールバックURL
   例えば、`GitHubProvider`プロバイダーを使用する場合、GitHubログインのURLは次のようになります：
3. `/api/auth/github/login` - ログインを開始するURL
4. `/api/auth/github/callback` - ログインを完了するためのコールバックURL 一般的なコールバックに渡された引数で`provider`を決定できます。

#### SSLプロキシの設定 {#ssl}

アプリがSSLプロキシ（Nginx）の背後にある場合、`secureProxy`オプションを`true`に設定する必要があります。プロキシは`forwarded`または`x-forwarded-proto`ヘッダーを正しく管理する必要があります。

```ts
// src/pages/api/auth/[...nextauth].ts
import { NextAuthAdapter } from "@blitzjs/auth/next-auth"
import { api } from "src/blitz-server"
import db from "db"
export default api(
  NextAuthAdapter({
    successRedirectUrl: "/",
    errorRedirectUrl: "/",
    secureProxy: true, // ハイライト行
    strategies: [
      /*...*/
    ],
  })
)
```

### 3. NextAuthプロバイダーの追加 {#2-add-a-next-auth-provider}

APIルートの`NextAuthAdapter`の`providers`配列引数にプロバイダーを追加し、その後の設定についてはプロバイダードキュメントに従ってください。

```ts
// src/pages/api/auth/[...nextauth].ts
import { api } from "src/blitz-server"
import GithubProvider from "next-auth/providers/github"
import { NextAuthAdapter, BlitzNextAuthOptions } from "@blitzjs/auth/next-auth"
import db, { User } from "db"
import { Role } from "types"

const config: BlitzNextAuthOptions = {
  successRedirectUrl: "/",
  errorRedirectUrl: "/",
  // ハイライト開始
  providers: [
    GithubProvider({
      clientId: process.env.GITHUB_CLIENT_ID as string,
      clientSecret: process.env.GITHUB_CLIENT_SECRET as string,
    }),
  ],
  callback: async (user, account, profile, session, provider) => {
    let newUser: User
    try {
      newUser = await db.user.findFirstOrThrow({
        where: { name: { equals: user.name } },
      })
    } catch (e) {
      newUser = await db.user.create({
        data: {
          email: user.email!,
          name: user.name || "unknown",
          role: "USER",
        },
      })
    }
    await session.$create({
      userId: newUser.id,
      role: newUser.role as Role,
      source: "github",
    })
    return { redirectUrl: "/github" }
    // 返り値がない場合、successRedirectUrlがデフォルトになります
  },
  // ハイライト終了
}
export default api(NextAuthAdapter(config))
```

注意: 上記の`GitHubProvider`の例では、`User` prismaモデルに`email String @unique`および`name String`が必要です。

### 3. このNextAuthプロバイダーでログイン {#3-log-in-with-this-next-auth-provider}

URL形式が`/api/auth/[providerName]/login`のリンクをアプリに追加します。上記のGitHubの例では、リンクは次のようになります：

```tsx
<a href="/api/auth/github">GitHubでログイン</a>
```

## 詳細な使用手順 {#detailed-usage-instructions}

サードパーティとの認証が成功すると、ユーザーは上記の認証APIルートにリダイレクトされます。そのとき、`verify`コールバックが呼び出されます。`verify`コールバックが呼び出されると、そのユーザーはサードパーティによって認証されていますが、**Blitzアプリのセッションはまだ作成されていません**。

### セッションを作成する {#create-a-session}

**新しいBlitzセッションを作成するには**、コールバック関数に渡される`session`引数を使用する必要があります。

```ts
session.$create({
    ...
})
```

### エラーを返す {#return-an-error}

代わりに、何らかのエラーのためにセッションを作成しないようにしたい場合は、コールバック関数内でエラーをスローしてください。Blitzはエラーをキャッチし、それを提供されたエラーリダイレクトURLに添付します。

```ts
throw new YourAuthFailureError()
```

### ユーザーにエラーを表示する {#showing-the-error-to-the-user}

このプロセス中のエラーは、`authError`クエリパラメータとして提供されます。例えば、`errorRedirectUrl = '/'` で`done(new Error("it broke"))`のとき、ユーザーは次のようにリダイレクトされます：

```
/?authError=it broke
```

### 認証後のリダイレクト {#post-authentication-redirects}

ユーザーが認証された後にリダイレクトされるURLを決定するための方法は、優先順位順に4つあります。方法#1で提供されたURLが他のすべてのURLを上書きします。

- プロバイダーに応じて、必要なURLに`redirectUrl`を返す

```ts
return { redirectUrl: "/github" }
```

- "ログインを開始"するURLに`redirectUrl`クエリパラメータを追加する
  - 例: `example.com/api/auth/github?redirectUrl=/dashboard`
  - 例: `example.com/api/auth/github?redirectUrl=${router.pathname}`

- `NextAuthAdapter`に渡される構成を介して
  - 成功した場合、`config.successRedirectUrl`が使用されます
  - エラーの場合、`config.errorRedirectUrl`が使用されます
