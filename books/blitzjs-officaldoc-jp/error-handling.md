---
title: エラーハンドリング
sidebar_label: エラーハンドリング
---

## 組み込みエラー {#built-in-errors}

Blitzには、アプリケーション全体で使用できる便利なエラーがいくつか用意されています。

- `AuthenticationError`
  - `name`: "AuthenticationError"
  - `statusCode`: 401
  - デフォルトの`message`: "アクセスするにはログインが必要です"
- `CSRFTokenMismatchError`
  - `name`: "CSRFTokenMismatchError"
  - `statusCode`: 401
  - デフォルトの`message`: "アクセスするにはログインが必要です"
- `AuthorizationError`
  - `name`: "AuthorizationError"
  - `statusCode`: 403
  - デフォルトの`message`: "アクセスする権限がありません"
- `NotFoundError`
  - `name`: "NotFoundError"
  - `statusCode`: 404
  - デフォルトの`message`: "見つかりません"
- `RedirectError`
  - `name`: "RedirectError"
  - このエラーをレンダリング関数からスローすると、ユーザーが現在のレンダリングパスを見ずにリダイレクトを行います。`ErrorBoundary`コンポーネントが自動的にこのリダイレクトを処理します。
  - 例: `throw new RedirectError('/login')`

使用するには、`blitz`からインポートし、任意のJavaScriptエラーと同様に使用します。興味がある場合は、[これらのソースコードを確認できます](https://github.com/blitz-js/blitz/blob/canary/packages/core/src/errors.ts)。

```ts
import { AuthenticationError } from "blitz"

try {
  throw new AuthenticationError()
} catch (error) {
  if (error.name === "AuthenticationError") {
    // このエラーを適切に処理する、例えばログイン画面を表示する
    console.log(error.statusCode)
    console.log(error.message)
  }
}
```

これらのエラーやその他のエラーは、アプリのどこからでも、サーバー側でもクライアント側でもスローすることができます。

## クライアントでのエラーのキャッチと処理 {#catching-and-handling-errors-on-the-client}

クライアントでエラーを処理するには、[`<ErrorBoundary>`](./error-boundary)を使用します。

デフォルトでは、新しいBlitzアプリケーションには`pages/_app.tsx`にトップレベルの`ErrorBoundary`と`FallbackComponent`が含まれています。

以下のような感じです：

```tsx
// app/pages/_app.tsx
import { AppProps, ErrorBoundary, ErrorComponent } from "@blitzjs/next"
import { useQueryErrorResetBoundary } from "@blitzjs/rpc"
import LoginForm from "app/auth/components/LoginForm"

export default function App({ Component, pageProps }: AppProps) {
  // Blitz useQueryフックがエラーバウンダリをリセットするたびに自動的にデータを再取得するようにします
  const { reset } = useQueryErrorResetBoundary()

  return (
    <ErrorBoundary FallbackComponent={RootErrorFallback} onReset={reset}>
      <Component {...pageProps} />
    </ErrorBoundary>
  )
}

function RootErrorFallback({ error, resetErrorBoundary }) {
  if (error.name === "AuthenticationError") {
    return <LoginForm onSuccess={resetErrorBoundary} />
  } else if (error.name === "AuthorizationError") {
    return (
      <ErrorComponent
        statusCode={error.statusCode}
        title="申し訳ありませんが、アクセス権がありません"
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

つまり、すべてのエラーは少なくともルートレベルでキャッチされます。しかし、よりローカライズされたエラーハンドリングのために、アプリの他の場所に`<ErrorBoundary>`を追加することもできます。これを行うには、コンポーネントで`useQueryErrorResetBoundary`を別途宣言し、それをローカルのErrorBoundaryに渡します。アプリツリーの下の方で`<ErrorBoundary>`によってエラーがキャッチされた場合、それを再スローしない限り、ルートErrorBoundaryには到達しません。

### クライアントでのサーバーエラーの処理 {#handling-server-errors-on-the-client}

Blitzの非常に素晴らしい機能は、Blitzクエリまたはミューテーションから任意のエラーをスローし、それをフロントエンドのErrorBoundaryでキャッチして処理できることです。

例えば、上記の`_app.tsx`を使用すると、Blitzクエリ内で`AuthenticationError`をスローし、クライアントでログイン画面が自動的に表示されます。これは、ルートErrorBoundaryが`error.name=== 'AuthenticationError'`の場合に`<LoginForm>`をレンダリングするからです。

### カスタムエラー {#custom-errors}

Blitzが提供するエラー以外の場合、カスタムエラークラスを作成することをお勧めします。次に、エラー処理を助けるためのカスタムデータ属性を追加できます。

以下はカスタムエラーの作成例です。これはJavaScriptクラスなので、自由に創造性を発揮できます。

```ts
import SuperJson from "superjson"

export class UsernameTakenError extends Error {
  name = "UsernameTakenError"
  constructor({ suggestedUserName }) {
    super()
    this.suggestedUserName = suggestedUserName
  }
}
// クライアントで再構築されるようにSuperJsonシリアライザに登録
SuperJson.registerClass(UsernameTakenError)
SuperJson.allowErrorProps("suggestedUserName")

throw new UsernameTakenError({ suggestedUserName: "second_best" })
```

上記のように、**SuperJsonに登録する必要がある**ことに注意してください。これにより、クライアントでの`instanceof`が機能します。また、シリアライズしたい特別なエラー属性をSuperJsonに通知する必要があります。デフォルトでは、カスタムエラー属性はセキュリティ上の理由から省略されます。

その後、クライアントでは、上記のような`FallbackComponent`を使用するか、以下のようにフォーム送信ハンドラでエラーを処理できます。

```tsx
<Form
  onSubmit={async (values) => {
    try {
      await setUsername(values.username)
    } catch (error) {
      if (error instanceof UsernameTakenError) {
        setSuggestedUsername(error.suggestedUserName)
      }
    }
  }}
/>
```
