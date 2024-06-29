---
title: 他のユーザーを偽装する方法
sidebar_label: 他のユーザーを偽装する方法
---

他のユーザーを偽装することは、SaaSアプリケーションにとって重要なツールであり、管理者が特定のユーザーの視点からアプリケーションを閲覧することを可能にします。これにより、管理者はそのユーザーの資格情報を知ることなく操作ができます。

最終的な結果がどのようになるかの例はこちらです:
![最終結果のスクリーンショット](https://pbs.twimg.com/media/EsWuHShXcAEgEtw?format=jpg&name=large)

最初に、`impersonatingFromUserId`型を[セッション](/docs/session-management#customize-session-public-data-in-typescript)に追加します。

```ts
// types.ts
import { SessionContext } from "@blitzjs/auth"
import { User } from "db"

import { SimpleRolesIsAuthorized } from "@blitzjs/auth"
import { User } from "db"

declare module "@blitzjs/auth" {
  export interface Session {
    isAuthorized: SimpleRolesIsAuthorized<Role>
    PublicData: {
    role: "admin" | "customer"
    userId: User["id"]
    orgId: User["orgId"]
    impersonatingFromUserId?: number
  }
}
```

次に、以下の2つのミューテーションを作成します。これにより、偽装の開始と停止が可能になります。

```ts
// src/auth/mutations/impersonateUser.ts
import { resolver } from "@blitzjs/rpc"
import db from "db"
import * as z from "zod"

export const ImpersonateUserInput = z.object({
  userId: z.number(),
})

export default resolver.pipe(
  resolver.zod(ImpersonateUserInput),
  resolver.authorize(),
  async ({ userId }, ctx) => {
    const user = await db.user.findFirst({ where: { id: userId } })
    if (!user) throw new Error("Could not find user id " + userId)

    await ctx.session.$create({
      userId: user.id,
      role: user.role,
      orgId: user.organizationId,
      impersonatingFromUserId: ctx.session.userId,
    })

    return user
  }
)
```

```ts
// src/auth/mutations/stopImpersonating.ts
import { resolver } from "@blitzjs/rpc"
import db from "db"

export default resolver.pipe(resolver.authorize(), async (_, ctx) => {
  const userId = ctx.session.$publicData.impersonatingFromUserId
  if (!userId) {
    console.log("Not impersonating anyone")
    return
  }

  const user = await db.user.findFirst({
    where: { id: userId },
  })
  if (!user) throw new Error("Could not find user id " + userId)

  await ctx.session.$create({
    userId: user.id,
    role: user.role,
    orgId: user.organizationId,
    impersonatingFromUserId: undefined,
  })

  return user
})
```

ユーザーを切り替えるためのフォームを以下のように追加します。

```tsx
import { useMutation, queryClient } from "@blitzjs/rpc"
import impersonateUser, {
  ImpersonateUserInput,
} from "src/auth/mutations/impersonateUser"
import { Form, FORM_ERROR } from "src/core/components/Form"
import LabeledTextField from "src/core/components/LabeledTextField"

export const ImpersonateUserForm = () => {
  const [impersonateUserMutation] = useMutation(impersonateUser)

  return (
    <Form
      schema={ImpersonateUserInput}
      submitText="Switch to User"
      onSubmit={async (values) => {
        try {
          await impersonateUserMutation(values)
          queryClient.clear()
        } catch (error) {
          return {
            [FORM_ERROR]:
              "Sorry, we had an unexpected error. Please try again. - " +
              error.toString(),
          }
        }
      }}
    >
      <LabeledTextField name="userId" type="number" label="User ID" />
    </Form>
  )
}
```

最後に、以下のコンポーネントをレイアウトの上部に追加します。

```tsx
// src/core/components/ImpersonatingUserNotice.tsx
import { invoke, useSession, queryClient } from "@blitzjs/rpc"
import stopImpersonating from "src/auth/mutations/stopImpersonating"

export const ImpersonatingUserNotice = () => {
  const session = useSession()
  if (!session.impersonatingFromUserId) return null

  return (
    <div className="bg-yellow-400 px-2 py-1 text-center font-semibold">
      現在ユーザー {session.userId} を偽装中{" "}
      <button
        className="appearance-none bg-transparent text-black uppercase"
        onClick={async () => {
          await invoke(stopImpersonating, {})
          queryClient.clear()
        }}
      >
        終了
      </button>
    </div>
  )
}
```

これで完了です！ユーザーを偽装し、その後再度停止することができるようになりました。
