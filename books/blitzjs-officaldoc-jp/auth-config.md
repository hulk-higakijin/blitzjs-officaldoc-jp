    ===
    ---
title: 認証設定
sidebar_label: 設定
---

`src/blitz-server` の `AuthServerPlugin()` にオブジェクトを渡すことで、セッション管理をカスタマイズできます。

```ts
// src/blitz-server.ts
import { AuthServerPlugin, PrismaStorage } from "@blitzjs/auth"
import { simpleRolesIsAuthorized } from "@blitzjs/auth"

setupBlitzServer({
  plugins: [
    AuthServerPlugin({
      cookiePrefix: "web-cookie-prefix",
      storage: PrismaStorage(db),
      isAuthorized: simpleRolesIsAuthorized,
    }),
  ],
})
```

利用可能なオプション:

```ts
type SessionConfigOptions = {
  cookiePrefix?: string /* デフォルト: 'blitz' */
  sessionExpiryMinutes?: number /* デフォルト: 30日 */
  anonSessionExpiryMinutes?: number /* デフォルト: 5年 */
  method?: "essential" | "advanced" /* デフォルト: 'essential' */
  sameSite?: "strict" | "lax" | "none" /* デフォルト: 'lax'. 
    これを 'none' に設定する場合は、`secureCookies: true` も設定する必要があります。
    参照: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite */
  secureCookies?: boolean /* デフォルト: undefined. アプリがホスト名 `localhost` で実行されている場合、このフラグはスキップされます */
  domain?: string /* デフォルト: undefined. サブドメイン間でも機能するように `.yourDomain.com` と設定可能 */
  publicDataKeysToSyncAcrossSessions?: string[] /* デフォルト: ['role', 'roles'] */
}
```

## セッションの永続化とデータベースアクセスのカスタマイズ {#customize-session-persistence-and-database-access}

デフォルトでは、セッションの永続化は `PrismaStorage` を使ったゼロ設定です。しかし、セッションをRedisなどの他の場所に保存するようにカスタマイズすることができます。Prismaを使用している場合でも、ユーザーやセッションモデルの属性名をカスタマイズしたい場合にこの方法が使えます。

データベースアクセス機能をオーバーライドしてセッションの永続化をカスタマイズします。この機能は何でも行うことができますが、定義された入力および出力タイプに準拠している必要があります。

```typescript
import { setupBlitzServer } from "@blitzjs/next"
import {
  AuthServerPlugin,
  PrismaStorage,
  simpleRolesIsAuthorized,
} from "@blitzjs/auth"

const { gSSP, gSP, api } = setupBlitzServer({
  plugins: [
    AuthServerPlugin({
      cookiePrefix: "blitz-app-prefix",
      isAuthorized: simpleRolesIsAuthorized,
      storage: {
        getSession: (handle) => redis.get(`session:${handle}`),
        createSession: (session) =>
          redis.set(`session:${handle}`, session),
        deleteSession: (handle) => redis.del(`session:${handle}`),
        // ...
      },
    }),
  ],
})

export { gSSP, gSP, api }
```

[**完全なRedisの例はこちらのGitHub Gistです。**](https://gist.github.com/beerose/80f37b4b36cbd7ba2745701959e3cb8b)

ストレージメソッドは以下のインターフェースに準拠する必要があります:

```typescript
interface SessionConfigMethods {
  getSession: (handle: string) => Promise<SessionModel | null>
  getSessions: (userId: PublicData["userId"]) => Promise<SessionModel[]>
  createSession: (session: SessionModel) => Promise<SessionModel>
  updateSession: (
    handle: string,
    session: Partial<SessionModel>
  ) => Promise<SessionModel | undefined>
  deleteSession: (handle: string) => Promise<SessionModel>
}

interface SessionModel extends Record<any, any> {
  handle: string
  userId?: PublicData["userId"] | null
  expiresAt?: Date | null
  hashedSessionToken?: string | null
  antiCSRFToken?: string | null
  publicData?: string | null
  privateData?: string | null
}
```

    ===