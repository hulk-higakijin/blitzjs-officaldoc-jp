```mdx
---
title: Passport.jsによるサードパーティログイン
sidebar_label: Passport.js wによるサードパーティログイン
---

Blitzは既存の[Passport.js認証戦略](http://www.passportjs.org/)を使用できるアダプターを提供します。

現在、`verify`コールバックを使用するpassport戦略のみがサポートされています。以下のTwitterの例では、`TwitterStrategy()`の2番目の引数が`verify`コールバックです。

## 設定 {#setup}

### 1. Passport.js APIルートを追加 {#1-add-the-passport-js-api-route}

以下の内容で`src/pages/api/auth/[...auth].ts`に新しいAPIルートを追加します。

```ts
// src/pages/api/auth/[...auth].ts
import { passportAuth } from "@blitzjs/auth"
import { api } from "src/blitz-server"
import db from "db"

export default api(
  passportAuth({
    successRedirectUrl: "/",
    errorRedirectUrl: "/",
    strategies: [
      {
        strategy: new PassportStrategy(), // ここに初期化されたパスポート戦略を指定
      },
    ],
  })
)
```

必要に応じて、APIルートを異なるパスに配置することもできますが、ファイル名は`[...auth].js`または`[...auth].ts`である必要があります。

#### URLs

`passportAuth`アダプターは、インストールされた各戦略のために2つのAPIエンドポイントを追加します。

ハンドラが`src/pages/api/auth/[...auth].ts`にある場合、以下が追加されます：

1. `/api/auth/[strategyName]` - ログインを開始するURL
2. `/api/auth/[strategyName]/callback` - ログインを完了するためのコールバックURL

例えば、`passport-twitter`戦略を使用する場合、TwitterのURLは以下のようになります：

1. `/api/auth/twitter` - ログインを開始するURL
2. `/api/auth/twitter/callback` - ログインを完了するためのコールバックURL

`strategyName`は戦略のドキュメント内の`passport.authenticate('github')`のような部分を確認することで特定できます。この場合、`strategyName`は`github`です。

#### SSLプロキシ設定

アプリがSSLプロキシ（Nginx）の背後にある場合、`secureProxy`オプションを`true`に設定する必要があるかもしれません。プロキシは`forwarded`または`x-forwarded-proto`ヘッダーを正しく管理するように設定する必要があります。

```ts
// src/pages/api/auth/[...auth].ts
import { passportAuth } from "@blitzjs/auth"
import { api } from "src/blitz-server"
import db from "db"

export default api(
  passportAuth({
    successRedirectUrl: "/",
    errorRedirectUrl: "/",
    secureProxy: true, // highlight-line
    strategies: [
      /*...*/
    ],
  })
)
```

#### ミドルウェアコンテキスト、リクエスト、レスポンスオブジェクトへのアクセス

`passportAuth`アダプターにコールバックを提供することで、ミドルウェアコンテキストおよびリクエストとレスポンスオブジェクトにアクセスできます。コールバックの引数は `ctx`、`req`、`res` プロパティを持つオブジェクトです。その後、パスポート戦略にカスタムパラメータ（例：招待コード、紹介コード）を含める必要がある場合、セッションコンテキストは `ctx.session` を介して、リクエストオブジェクトにアクセスすることができます。

```ts
// src/pages/api/auth/[...auth].ts
import { passportAuth } from "@blitzjs/auth"
import { api } from "src/blitz-server"
import db from "db"

export default api(
  passportAuth(({ ctx, req, res }) => ({
    successRedirectUrl: "/",
    errorRedirectUrl: "/",
    strategies: [
      {
        strategy: new TwitterStrategy({
          consumerKey: process.env.TWITTER_CONSUMER_KEY as string,
          consumerSecret: process.env.TWITTER_CONSUMER_SECRET as string,
          /*...*/
        }),
      },
    ],
  }))
)
```

注：環境変数に型が付けられていない場合、コールバック使用時に各環境変数に型アサーションを追加する必要があります（上記の例のように）。

### 2. パスポート戦略を追加 {#2-add-a-passport-strategy}

APIルート内で`passportAuth`の引数`strategies`配列に戦略を追加し、その後戦略のドキュメントに従って設定します。

ここでは、`passport-twitter`を追加する例を示します。

`callbackURL`が上記で説明したコールバックエンドポイント（`/api/auth/twitter/callback`）を使用することに注意してください。

```ts
import { passportAuth } from "@blitzjs/auth"
import { api } from "src/blitz-server"
import db from "db"
import { Strategy as TwitterStrategy } from "passport-twitter"

export default api(
  passportAuth({
    successRedirectUrl: "/",
    errorRedirectUrl: "/",
    strategies: [
      // highlight-start
      {
        strategy: new TwitterStrategy(
          {
            consumerKey: process.env.TWITTER_CONSUMER_KEY,
            consumerSecret: process.env.TWITTER_CONSUMER_SECRET,
            callbackURL:
              process.env.NODE_ENV === "production"
                ? "https://example.com/api/auth/twitter/callback"
                : "http://localhost:3000/api/auth/twitter/callback",
            includeEmail: true,
          },
          async function (_token, _tokenSecret, profile, done) {
            const email = profile.emails && profile.emails[0]?.value

            if (!email) {
              // これはTwitterアプリの権限でメールアクセスを有効にしていない場合に発生することがあります
              return done(
                new Error("Twitter OAuth response doesn't have email.")
              )
            }

            const user = await db.user.upsert({
              where: { email },
              create: {
                email,
                name: profile.displayName,
              },
              update: { email },
            })

            const publicData = {
              userId: user.id,
              roles: [user.role],
              source: "twitter",
            }
            done(undefined, { publicData })
          }
        ),
      },
      // highlight-end
    ],
  })
)
```

注：上記の`passport-twitter`の例では`User` prismaモデルに`email String @unique`および`name String`が必要です。

### 3. このパスポート戦略でログイン {#3-log-in-with-this-passport-strategy}

あなたのアプリに`/api/auth/[strategyName]`形式のURLのリンクを追加します。

上記のTwitterの例では、リンクは次のようになります：

```tsx
<a href="/api/auth/twitter">Log In With Twitter</a>
```

## 詳細な使用手順 {#detailed-usage-instructions}

サードパーティでの認証に成功すると、ユーザーは上記の認証APIルートにリダイレクトされます。その際、`verify`コールバックが呼び出されます。

`verify`コールバックが呼び出されると、ユーザーはサードパーティで認証されますが、**Blitzアプリのセッションはまだ作成されていません**。

### セッションを作成 {#create-a-session}

**新しいBlitzセッションを作成するには**、`verify`コールバックから`done()`関数を呼び出す必要があります。

```ts
done(undefined, result)
```

ここで`result`は`VerifyCallbackResult`型のオブジェクトです。

```ts
export type VerifyCallbackResult = {
  publicData: PublicData
  privateData?: Record<string, any>
  redirectUrl?: string
}
```

Blitzアダプターはその後、`session.$create()`を呼び出し、ユーザーをアプリケーション内の正しい場所にリダイレクトします。

### エラーを返す {#return-an-error}

代わりに、何らかのエラーのためにセッションの作成を防ぎたい場合は、最初の引数としてエラーを指定して`done()`を呼び出します。その後、ユーザーは正しい場所にリダイレクトされます。

```ts
return done(new Error("it broke"))
```

### ユーザーにエラーを表示 {#showing-the-error-to-the-user}

このプロセス中のいかなるエラーも、`authError`クエリパラメータとして提供されます。

例えば、`errorRedirectUrl = '/'`と`done(new Error("it broke"))`の場合、ユーザーは以下にリダイレクトされます：

```
/?authError=it broke
```

### 認証後のリダイレクト {#post-authentication-redirects}

ユーザーが認証された後にリダイレクトされるURLを決定する方法は4つあります。以下に優先順位でリストされています。方法#1で提供したURLは他のすべてのURLをオーバーライドします。

1. `verify`コールバック結果に`redirectUrl`を追加
   - 例: `done(undefined, {publicData, redirectUrl: '/'})`
2. "ログインを開始"するURLに`redirectUrl`クエリパラメータを追加
   - 例: `example.com/api/auth/twitter?redirectUrl=/dashboard`
   - 例: `example.com/api/auth/twitter?redirectUrl=${router.pathname}`
3. `passportAuth`に渡された設定経由
   - 成功であれば、`config.successRedirectUrl`を使用
   - エラーであれば、`config.errorRedirectUrl`を使用
4. 上記のいずれも提供されない場合、`/`にリダイレクト

注：エラーが発生した場合、方法#1および#2は`config.errorRedirectUrl`をオーバーライドします。

これで最大限の柔軟性が提供されます。これでもニーズを満たしていない場合は、GitHubで問題を報告してください。

### `authenticateOptions` {#authenticate-options}

一部の戦略では、`passport.authenticate()`メソッド内で`scope`や`successMessage`のようなオプションを呼び出す必要があります。これらのオプションを以下のように`passportAuth`オブジェクトに追加します：

```ts
import { passportAuth } from "@blitzjs/auth"
import { api } from "src/blitz-server"
import db from "db"
import { Strategy as Auth0Strategy } from "passport-auth0"

export default api(
  passportAuth({
    successRedirectUrl: "/",
    errorRedirectUrl: "/",
    strategies: [
      {
        // highlight-start
        authenticateOptions: { scope: "openid email profile" },
        // highlight-end
        strategy: new Auth0Strategy(
          {
            domain: process.env.AUTH0_DOMAIN,
            clientID: process.env.AUTH0_CLIENT_ID,
            clientSecret: process.env.AUTH0_CLIENT_SECRET,
            callbackURL:
              process.env.NODE_ENV === "production"
                ? "https://example.com/api/auth/auth0/callback"
                : "http://localhost:3000/api/auth/auth0/callback",
          },
          async function (
            _token,
            _tokenSecret,
            extraParams,
            profile,
            done
          ) {
            const email = profile.emails && profile.emails[0]?.value

            if (!email) {
              // これはAuth0アプリの権限でメールアクセスを有効にしていない場合に発生することがあります
              return done(new Error("Auth response doesn't have email."))
            }

            const user = await db.user.upsert({
              where: { email },
              create: {
                email,
                name: profile.displayName,
              },
              update: { email },
            })

            const publicData = {
              userId: user.id,
              roles: [user.role],
              source: "auth0",
            }
            done(undefined, { publicData })
          }
        ),
      },
    ],
  })
)
```

注：`authenticateOptions`がないと、`verify`関数内の`profile`パラメータには値が含まれません。

### カスタム戦略名 {#custom-strategy-name}

認証APIルートを作成する際、Blitzはデフォルトの戦略名を使用します。例えば、Twitter戦略の場合、APIルートは`/api/auth/twitter`および`/api/auth/twitter/callback`になります。名前を上書きしてカスタムのものを提供したい場合、`name`パラメータを指定できます：

```ts
export default api(
  passportAuth({
    successRedirectUrl: "/",
    errorRedirectUrl: "/",
    strategies: [
      {
        name: "custom-name", // highlight-line
        strategy: new TwitterStrategy(
          // ...
```

上記の設定は`example.com/api/auth/custom-name`および`api/auth/custom-name/callback`APIルートを生成します。
```