---
title: 設定
sidebar_label: 設定
---

`next.config.js`ファイル内のblitzプロパティ内で特定のRPCアスペクトを構成することができます。

## resolverPathオプションの変更 {#custom-resolver-path}

すべてのファイルパスの解決方法を変更するには、`next.config.js`内に`resolverPath`オプションを設定できます。

```ts
module.exports = withBlitz({
  blitz: {
    resolverPath: "queries|mutations",
  },
})

// またはカスタム関数を使用
module.exports = withBlitz({
  blitz: {
    resolverPath: (filePath) => {
      return filePath.replace("app/", "") // パスから`app/`を削除します
    },
  },
})
```

ビルド中にRPCパスを解決する方法を決定するための3つのオプションがあります。

- `queries|mutations` (デフォルト）
  - `queries`または`mutations`フォルダーをルートとして使用
- `root`
  - プロジェクトのルートから始まるパスを使用
- カスタム関数
  - 解決が必要なすべてのファイルパスに対してこの関数が呼ばれます。

出力例：

```
ファイル: src/products/queries/getProduct.ts
"queries|mutations" URL: /api/rpc/getProduct
"root" URL: src/products/queries/getProduct

ファイル: src/products/mutations/createProduct.ts
"queries|mutations" URL: /api/rpc/createProduct
"root" URL: src/products/mutations/createProduct

ファイル: src/products/mutations/v2/createProduct.ts
"queries|mutations" URL: /api/rpc/v2/createProduct
"root" URL: src/products/mutations/v2/createProduct
```

## 別フォルダーにあるリゾルバー {#resolvers-separate-folder}

`blitz`プロパティ内にBlitzのルートフォルダーからの相対パスフォルダーの配列として`includeRPCFolders`パラメーターを追加します。例えば：

```ts
module.exports = withBlitz({
  blitz: {
    includeRPCFolders: ["../../<YOUR DIRECTORY PATH>"],
  },
})
```

:::alert
リゾルバーは依然として`queries`または`mutations`フォルダーに配置する必要があります。
:::

## ロギング設定 {#blitz-rpc-logging}

`[[...blitz]].ts`APIファイル内の`rpcHandler`に以下の設定が表示されます:
```ts
logging?: {
  /**
    * allowListはロギングを有効にするルートのリストを表します
    * whiteListが定義されている場合、指定されたルートのみがログに記録されます
    */
  allowList?: string[]
  /**
    * blockListはロギングを無効にするルートのリストを表します
    * blockListが定義されている場合、指定されたルート以外がログに記録されます
    */
  blockList?: string[]
  /**
    * verboseはロギングを有効/無効にするフラグを表します
    * verboseがtrueの場合、Blitz RPCは各リゾルバーの入力と出力をログに記録します
    */
  verbose?: boolean
  /**
    * disablelevelは特定のレベルのロギングを有効/無効にするフラグを表します
    */
  disablelevel?: "debug" | "info"
}
```

:::alert
  Blitz RPCのデフォルト設定は：
  - `verbose`は構成されていない場合trueになります。
  - `info`レベルで`入力`と`リゾルバー完了時間`がログに記録されます。
  - `debug`レベルで`結果`と`next.jsのシリアル化時間`がログに記録されます。
:::

例：

```ts
export default api(
  rpcHandler({
    onError: console.log,
    formatError: (error) => {
      error.message = `FormatError handler: ${error.message}`
      return error
    },
    logging: {
      verbose: true,
      blockList: ["getCurrentUser", ...], //リゾルバ名（リゾルバファイル名）を記載するだけです
    },
  })
)
```

これは`getCurrentUser`およびその他の指定された`blockList`内のリゾルバー以外のすべてのリゾルバーに対して詳しいBlitz RPCロギングを有効にします。
