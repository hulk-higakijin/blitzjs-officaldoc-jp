    ===
    ---
    title: セットアップ
    sidebar_label: セットアップ
    ---
    
    1. 以下のコマンドで `@blitzjs/rpc` プラグインをインストールします:
    
    ```bash
    npm i @blitzjs/rpc # yarn add @blitzjs/rpc # pnpm add @blitzjs/rpc
    ```
    
    2. 以下の内容を `blitz-client.ts` ファイルに追加します:
    
    ```typescript
    import { setupClient } from "@blitzjs/next"
    import { BlitzRpcPlugin } from "@blitzjs/rpc"
    
    const { withBlitz } = setupClient({
      plugins: [BlitzRpcPlugin()],
    })
    
    export { withBlitz }
    ```
    
    `@blitzjs/rpc` プラグインを設定して、異なる `react-query` オプションを使用することもできます:
    
    ```typescript
    import { setupClient } from "@blitzjs/next"
    import { BlitzRpcPlugin } from "@blitzjs/rpc"
    
    const { withBlitz } = setupClient({
      plugins: [
        BlitzRpcPlugin({
          reactQueryOptions: {
            queries: {
              staleTime: 7000,
            },
          },
        }),
      ],
    })
    
    export { withBlitz }
    ```
    
    `react-query` の `QueryClient` オプションについてさらに知りたい場合は、
    [こちら](https://react-query.tanstack.com/reference/QueryClient)を参照してください。
    
    3. Blitz RPC API ルートを追加します:
    
    `src` ディレクトリ内に `pages/api/rpc` ディレクトリを作成し、`[[...blitz]].ts` ファイルを作成して
    次の行を追加します:
    
    ```ts
    // src/pages/api/rpc/[[...blitz]].ts
    
    import { rpcHandler } from "@blitzjs/rpc"
    import { api } from "src/blitz-server"
    
    export default api(rpcHandler({}))
    ```
    
    ---
    
    `@blitzjs/rpc` プラグインの機能を使用する方法については、[Query Resolvers](/docs/query-resolvers) と
    [Mutation Resolvers](/docs/mutation-resolvers) のドキュメントを参照してください。
    
    ===