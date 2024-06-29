---
title: クライアントサイドAPIの認証
---

## `useSession()` {#use-session}

`useSession(options?) => Partial<PublicData> & {isLoading: boolean}`

:::alert
  注意: `useSession()`はデフォルトでsuspenseを使用するため、ツリーの上位に`<Suspense>`コンポーネントが必要です。 もしくは`useSession({suspense: false})`を設定してsuspenseを無効にすることができます。
:::

### 例 {#example}

```ts
import { useSession } from "@blitzjs/auth"

function SomeComponent() {
  const session = useSession()

  session.userId
  session.role

  return /*... */
}
```

### 引数 {#arguments}

- `options:`
  - オプション
  - `initialPublicData: PublicData` - サーバーセッションから公開データを設定するためにSSRと共に使用します
  - `suspense: boolean` - デフォルトは`true`

### 戻り値 {#returns}

- `session: Partial<PublicData> & {isLoading: boolean}`

## `useAuthenticatedSession()` {#use-authenticated-session}

`useAuthenticatedSession(options?) => PublicData & {isLoading: boolean}`

ユーザーがログインしていない場合、`AuthenticationError`をスローします

### 例 {#example-1}

```ts
import { useAuthenticatedSession } from "@blitzjs/auth"

const session = useAuthenticatedSession()
```

### 引数 {#arguments-1}

- `options:`
  - オプション
  - `initialPublicData: PublicData` - サーバーセッションから公開データを設定するためにSSRと共に使用します
  - `suspense: boolean` - デフォルトは`true`

### 戻り値 {#returns-1}

- `session: PublicData & {isLoading: boolean}`

## `useAuthorize()` {#use-authorize}

`useAuthorize() => void`

ユーザーがログインしていない場合、`AuthenticationError`をスローします

### 例 {#example-2}

```ts
import { useAuthorize } from "@blitzjs/auth"

useAuthorize()
```

### 引数 {#arguments-2}

- なし

### 戻り値 {#returns-2}

- なし

## `useRedirectAuthenticated()` {#use-redirect-authenticated}

`useRedirectAuthenticated(to: string) => void`

これはログイン済みユーザーを指定されたURLパスにリダイレクトします。ログアウトユーザーには何も行いません。

### 例 {#example-3}

```ts
import { useRedirectAuthenticated } from "@blitzjs/auth"

useRedirectAuthenticated("/dashboard")
```

### 引数 {#arguments-3}

- `to: string`
  - **必須**
  - ログイン済みユーザーをリダイレクトするURLパス

### 戻り値 {#returns-3}

- なし
