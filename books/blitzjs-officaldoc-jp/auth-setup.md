    ===
    ---
    title: セットアップ
    sidebar_label: セットアップ
    ---

    `@blitzjs/auth`プラグインをインストールするには以下のコマンドを実行します：

    ```bash
    npm i @blitzjs/auth # yarn add @blitzjs/auth # pnpm add @blitzjs/auth
    ```

    ### クライアントセットアップ {#client-setup}

    `blitz-client.ts`ファイルに以下を追加します：

    ```typescript
    import { AuthClientPlugin } from "@blitzjs/auth"
    import { setupBlitzClient } from "@blitzjs/next"

    export const authConfig = {
      cookiePrefix: "testapp",
    }

    const { withBlitz } = setupBlitzClient({
      plugins: [AuthClientPlugin(authConfig)],
    })

    export { withBlitz }
    ```

    ### サーバーセットアップ {#server-setup}

    次に、`blitz-server.ts`ファイルに以下を追加します：

    ```typescript
    import { setupBlitzServer } from "@blitzjs/next"
    import {
      AuthServerPlugin,
      PrismaStorage,
      simpleRolesIsAuthorized,
    } from "@blitzjs/auth"
    import { db } from "db"
    import { authConfig } from "./blitz-client"

    const { gSSP, gSP, api } = setupBlitzServer({
      plugins: [
        AuthServerPlugin({
          ...authConfig,
          storage: PrismaStorage(db),
          isAuthorized: simpleRolesIsAuthorized,
        }),
      ],
    })

    export { gSSP, gSP, api }
    ```

    ## 本番環境のデプロイ要件 {#production-deployment-requirements}

    本番環境では、少なくとも32文字の`SESSION_SECRET_KEY`環境変数を提供する必要があります。
    これはJWTトークンを署名するための秘密鍵です。

    macOSおよびLinuxでは、ターミナルで`openssl rand -hex 16`を実行して生成できます。
    ===