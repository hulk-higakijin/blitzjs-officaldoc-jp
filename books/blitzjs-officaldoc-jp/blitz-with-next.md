---
title: 既存のNext.jsプロジェクトにBlitz.jsを追加する
---

既存のNext.jsプロジェクトがあり、Blitz Toolkitの一部または全てを使用したい場合、このページではその設定方法について説明します。いくつかのステップが必要で、それらを一つずつ見ていきます。

## 必要な依存関係を追加する {#adding-dependencies}

```sh
yarn add blitz @blitzjs/next

# または

pnpm add blitz @blitzjs/next

# または

npm i blitz @blitzjs/next
```

## Blitzサーバーのセットアップ {#blitz-server-setup}

Blitzの認証、ミドルウェア、RPCなどのサーバー機能を使用したい場合は、プロジェクト内のどこかに`blitz-server.ts`ファイルを作成する必要があります。例えば、`src/blitz-server.ts`に作成します。プラグインの追加方法については後で説明します。

```ts
import { setupBlitzServer } from "@blitzjs/next"

const {
  /* プラグインのエクスポート */
} = setupBlitzServer({
  plugins: [
    // ここにプラグインが入ります
  ],
})
```

## Blitzクライアントのセットアップ {#blitz-client-setup}

次に、Blitzのクライアント機能を使用したい場合は、`blitz-client.ts`ファイルを作成する必要があります。それは`blitz-server.ts`と同じ場所に置くことができます。例えば、`src/blitz-client.ts`です。

```ts
import { setupBlitzClient } from "@blitzjs/next"

export const { withBlitz } = setupBlitzClient({
  plugins: [
    // ここにプラグインが入ります
  ],
})
```

`withBlitz`関数は、Blitzのクライアントサイド機能でコンポーネントをラップするために必要です。

## Appコンポーネントで`withBlitz`を使用する {#use-withblitz-in-app-component}

クライアントでBlitzを使用するには、Appコンポーネントで`withBlitz`関数を使用する必要があります。

```ts
import { withBlitz } from "src/blitz-client"

function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}

export default withBlitz(App)
```

## `next.config.js`ファイルの修正 {#modifying-next-config-file}

Next.jsではページの場所を手動で入力する必要があります。Blitzには[ルートマニフェスト](./route-manifest)が付属しているので、以下のようにできます。

```tsx
<Link href={Routes.ProductsPage({ productId: 123 })} />
// 代わりに
<Link href={`/products/${123}`} />
```

これを有効にするには、`next.config.js`ファイルで設定を`withBlitz`でラップする必要があります。

```js
const { withBlitz } = require("@blitzjs/next")

module.exports = withBlitz()
```

## プラグインの追加 {#adding-plugins}

基本設定が完了したら、アプリで使用したいプラグインを追加できます。いくつかチェックする場所があります。

1. [`@blitzjs/auth`](./auth-setup) — 認証プラグインの設定方法やBlitzの認証システムの使用方法について説明しています。
2. [`@blitzjs/rpc`](./rpc-setup) — BlitzのゼロAPIレイヤーの設定方法やその詳細について学ぶことができます。

最後に、[`@blitzjs/next`アダプター](./blitzjs-next)に関する詳細情報を確認し、`getServerSideProps`、`getStaticProps`、Next APIルート、その他の場所でBlitzの機能を使用する方法について学びましょう。
