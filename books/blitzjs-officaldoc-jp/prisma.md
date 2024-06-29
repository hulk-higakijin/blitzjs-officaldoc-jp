    ===
    ---
    title: Prismaユーティリティ
    sidebar_label: Prismaユーティリティ
    ---

    ### `enhancePrisma(PrismaClient)` {#enhance-prisma-prisma-client}

    `enhancePrisma`は、Blitzアプリと統合するための小さなラッパーで、
    アプリケーションをより便利にします。

    - アプリケーションでPrismaクライアントのインスタンスが一つだけ
      になるようにし、データベース接続の数を減らします。ただし、
      サーバーレスデプロイメントの場合は、サーバーレス関数ごとに
      一つのインスタンスが確保されます。
    - Prismaクライアントがクライアントバンドルで使用されないようにします。
    - テストで使用するために`db.$reset() => Promise<void>`関数を追加します。
      例えば以下のように使用します：
      ```ts
      beforeEach(async () => {
        await db.$reset()
      })
      ```
    - [Blitz vitest プリセット](./testing)を使用している場合、テストの終了時に
      自動的にデータベース接続を閉じます。

    #### 例の使用法

    ```ts
    import { enhancePrisma } from "blitz"
    import { PrismaClient } from "@prisma/client"

    const EnhancedPrisma = enhancePrisma(PrismaClient)

    export * from "@prisma/client"
    export default new EnhancedPrisma()
    ```

    ===