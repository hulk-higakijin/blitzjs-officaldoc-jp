---
title: クライアントプラグイン
---

### 概要 {#overview}

プラグインはBlitzツールキットをカスタム機能で拡張するために使用され、`blitz`パッケージの`createClientPlugin`関数を使用して定義されます。

プラグインは次のように使用されます：

- `events`オブジェクトには、プラグインがリスニングできるイベントフックが含まれています。
  - Blitzは各`イベント`についてグローバルな`イベントリスナー`を生成し、これを使用してフックをトリガーします。
  - プラグインはイベントがトリガーされたときに必要なアクションを実行できます。
  - 各プラグイン`イベント`によって定義されたコードは`並列`で実行されます。

- `middleware`オブジェクトには、プラグインが定義できるミドルウェアフックが含まれています。
  - プラグインはミドルウェアが呼び出されたときに必要なアクションを実行できます。
  - 各プラグイン`ミドルウェア`によって定義されたコードは`パイプ`を通じて連続的に実行され、1つのミドルウェアの出力は次のミドルウェアの入力として渡されます。
  - ミドルウェアの実行順は、`src/blitz-client`ファイルでプラグインが定義された順序によって定義されます。

- `withProvider`関数は、Blitzラッパー`withBlitz`を通じてアプリのルートコンポーネントをラップするために使用されます。

- `exports`オブジェクトは、プラグインが`src/blitz-client`ファイルからインポートしてアプリケーションに公開する必要があるか、関数や変数をエクスポートするために使用されます。

### APIリファレンス {#api-reference}

- `createClientPlugin` - `(options: TPluginOptions) => ClientPlugin<TPluginExports>`

  - `events`
    - `onSessionCreated` - `() => Promise<void>`
      - 新しいセッションが作成されたときに呼び出されます。
    - `onRpcError` - `(error: Error) => Promise<void>`
      - RPCエラーが発生したときに呼び出されます。
  - `middleware`
    - `beforeHttpRequest` - `(request: RequestInit) => RequestInit`
      - RPCリクエストが行われる前に呼び出され、リクエストパラメータを変更できます。
    - `beforeHttpResponse` - `(response: Response) => Response`
      - RPCレスポンスが返される前に呼び出され、レスポンスを変更できます。
  - `withProvider` - `(component: ComponentType<TProps>) => { (props: TProps): JSX.Element displayName: string }`

  - `exports` - `TPluginExports extends object`

### Blitzが使用するミドルウェアとイベントフック {#middleware-and-events-used-by-blitz}

- イベント
  - `onSessionCreated`は、新しいセッションが`Blitz Auth`によって作成されたときに`React Query`クライアントをリセットするために`Blitz RPC`によって使用されます。
  - `onRpcError`は、レスポンスフェーズ中にエラーを処理するために`Blitz RPC`によって使用されます。
- ミドルウェア
  - `beforeHttpRequest`は、リクエストが`Blitz RPC`によって送信される前に必要な認証ヘッダーを追加するために`Blitz Auth`によって使用されます。
  - `beforeHttpResponse`は、レスポンスが`Blitz RPC`によって返される前に認証関連のロジックを処理するために`Blitz Auth`によって使用されます。

### クライアントプラグインの作成 {#creating-a-blitz-client-plugin}

クライアントプラグインを作成するためには、新しいファイルを作成します。例えば、`src/custom-plugin/plugin.tsx`で次のコードを追加します：

```tsx
// src/custom-plugin/plugin.tsx
import { createClientPlugin } from "blitz"
import { BlitzPage } from "@blitzjs/next"
import { ComponentProps, ComponentType } from "react"

type CustomPluginOptions = {
  // ... オプションの型定義
}

function withCustomProvider<TProps = any>(
  Page: ComponentType<TProps> | BlitzPage<TProps>
) {
  const CustomProviderRoot = (props: ComponentProps<any>) => {
    // ... カスタムルートコンポーネント
    return <Page {...props} />
  }
  for (let [key, value] of Object.entries(Page)) {
    CustomProviderRoot[key] = value
  }
  if (process.env.NODE_ENV !== "production") {
    CustomProviderRoot.displayName = `CustomProviderRoot`
  }
  return CustomProviderRoot
}

export const BlitzCustomPlugin = createClientPlugin<
  CustomPluginOptions,
  {}
>((options?: CustomPluginOptions) => {
  // ... プラグインコード
  return {
    events: {
      onSessionCreated: async () => {
        // 新しいセッションが作成されたときに呼び出されます
        // 通常はユーザーがログインまたはログアウトするとき
      },
      onRpcError: async (error) => {
        // RPCコールが失敗したときに呼び出されます
      },
    },
    middleware: {
      beforeHttpRequest: (req) => {
        // RPCコール前にリクエストオプションを変更
        return req
      },
      beforeHttpResponse: (res) => {
        // 呼び出し元に返す前にレスポンスを変更
        return res
      },
    },
    exports: () => ({
      // ... プラグインエクスポート
    }),
    withProvider: withCustomProvider,
  }
})
```

### クライアントプラグインの使用 {#using-a-blitz-client-plugin}

作成したクライアントプラグインを初期化するには、`src/blitz-client`にインポートして次のように`plugins`配列に追加します：

```ts
// src/blitz-client.ts
import { AuthClientPlugin } from "@blitzjs/auth"
import { setupBlitzClient } from "@blitzjs/next"
import { BlitzRpcPlugin } from "@blitzjs/rpc"
import { BlitzCustomPlugin } from "./custom-plugin/plugin"

export const { withBlitz } = setupBlitzClient({
  plugins: [
    AuthClientPlugin({}),
    BlitzRpcPlugin({}),
    BlitzCustomPlugin({}), // <-- ここにプラグインを追加
  ],
})
```

### 例 {#example}

以下の例は、RPCリクエストにカスタムヘッダーを追加するプラグインを作成する方法を示しています：

```ts
export const BlitzCustomPlugin = createClientPlugin<
  CustomPluginOptions,
  {}
>((options?: CustomPluginOptions) => {
  // ... プラグインコード
  return {
    middleware: {
      beforeHttpRequest: (req) => {
        req.headers = {
          ...req.headers,
          "X-Custom-Header": "Custom Header Value",
        }
        return req
    },
    exports: () => ({
      // ... プラグインエクスポート
    }),
  }
})
```
