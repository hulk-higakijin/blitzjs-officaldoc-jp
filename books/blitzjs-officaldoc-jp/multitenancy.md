```mdx
---
title: マルチテナンシー
sidebar_label: マルチテナンシー
---

マルチテナンシーとは、1つのアプリが複数のユーザーや組織にサービスを提供し、そのデータがシステムの他のユーザーからプライベートに保たれるソフトウェアアーキテクチャのことです。この方法の1つは、各ユーザーに対して完全に別々のデータベースを持つことですが、これは高い運用オーバーヘッドを伴います。最も一般的な方法は、すべてのユーザーデータを1つのデータベースに格納し、外部キーを使用してデータをプライベートに保つことです。

このガイドでは、マルチテナントBlitzアプリを実装するための良い方法を紹介します。

## データモデル {#data-model}

データモデルを[Bullet TrainのAndrew Culverが説明した通りに](https://blog.bullettrain.co/teams-should-be-an-mvp-feature/)実装することをお勧めします。

- `Organization`はアカウントのすべてを所有する「神」モデルです
- `Organization`は`Membership`を通じて**多くの**`User`を**持っています**
- システム内の他のすべてのモデルには、誰が所有しているかを示す`organizationId`があります
- `User`は複数の`Organization`にアクセスできます
- タスクのようにエンティティをユーザーに割り当てる場合は、直接ユーザーに割り当てるのではなく、ユーザーの`Membership`にタスクを割り当てます。詳しくは上記のBullet Trainのブログ投稿を参照してください。

Prismaスキーマは次のようになります:

```prisma
model Organization {
  id         Int @id @default(autoincrement())
  name       String

  membership Membership[]
}

model Membership {
  id             Int @id @default(autoincrement())
  role           MembershipRole

  organization   Organization @relation(fields: [organizationId], references: [id])
  organizationId Int

  user           User? @relation(fields: [userId], references: [id])
  userId         Int?

  // When the user joins, we will clear out the name and email and set the user.
  invitedName  String?
  invitedEmail String?

  @@unique([organizationId, invitedEmail])
}

enum MembershipRole {
  OWNER
  ADMIN
  USER
}

// The owners of the SaaS (you) can have a SUPERADMIN role to access all data
enum GlobalRole {
  SUPERADMIN
  CUSTOMER
}

model User {
  id             Int      @id @default(autoincrement())
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
  name           String?
  email          String   @unique
  role           GlobalRole

  memberships Membership[]
}
```

その後、ユーザを作成すると同時に組織とメンバーシップも作成するように`signup`ミューテーションを更新する必要があります。次のようにします:

```ts
const user = await db.user.create({
  data: {
    // ...
    memberships: {
      create: {
        role: "OWNER",
        organization: {
          create: {
            name: organizationName,
          },
        },
      },
    },
  },
  include: { memberships: true },
})
```

## ユーザーセッション {#sessions}

上記のデータモデルでは、1人のユーザーが複数の組織に所属することが可能です。それでは、どうやってユーザーが現在アクセスまたは変更している組織を追跡するのでしょうか？

最も良い方法は、ユーザーが一度に1つの組織にしかアクセスしないようにすることです。そして、UIに彼らがアクセスする組織を切り替えるメニューを提供します。

そのためには、まず[セッションの`PublicData`](./session-management#change-session-public-data-of-current-user)に`orgId`フィールドを追加します。

`types.ts`を次のように更新します:

```diff
+import { GlobalRole, MembershipRole, Organization } from "db"

-export type Role = "ADMIN" | "USER"
+ type Role = MembershipRole | GlobalRole

declare module "@blitzjs/auth" {
  export interface Session {
    isAuthorized: SimpleRolesIsAuthorized<Role>
    PublicData: {
       userId: User["id"]
+      roles: Array<Role>
+      orgId?: Organization["id"]
    }
  }
}
```

次に、`ctx.session.$create()`を呼び出しているすべての場所を更新します。`orgId`を追加し、`roles`を更新する必要があります。

次のようになります:

```ts
await session.$create({
  userId: user.id,
  roles: [user.role, user.memberships[0].role],
  orgId: user.memberships[0].organizationId,
})
```

その後、クエリとミューテーションで`ctx.session.orgId`を使用して、現在の組織に基づいてクエリをフィルタリングできます。

## クエリ {#queries}

すべてのクエリを`organizationId`でフィルタリングすることで、1人のユーザーが他のユーザーのプライベートデータを閲覧できないようにする必要があります。

```ts
import db from "db"

// If you accept only the `id` as input
const project = await db.project.findFirst({
  where: {
    id: input.id,
    organizationId: ctx.session.orgId,
  },
})

// If you accept `where` as input
const projects = await db.project.findMany({
  where: {
    ...input.where,
    organizationId: ctx.session.orgId,
  },
})
```

## ミューテーション {#mutations}

新しいエンティティを作成する場合は、現在の組織にそれらを付けることを確認してください。次のようにします:

```ts
import { resolver } from "@blitzjs/rpc"
import db from "db"
import * as z from "zod"

const CreateProject = z
  .object({
    name: z.string(),
  })
  .nonstrict()

export default resolver.pipe(
  resolver.zod(CreateProject),
  resolver.authorize(),
  async (input, ctx) => {
    const project = await db.project.create({
      data: {
        ...input,
        organizationId: ctx.session.orgId,
      },
    })

    return project
  }
)
```

また、他のユーザーのデータが変更されないように、更新および削除ミューテーションも`organizationId`でフィルタリングする必要があります。

次に、他の組織に属するエンティティのIDを受け取る可能性があるミューテーションの例を示します。最初にそのIDに対して`db.project.findFirst()`クエリを実行し、手動で`organizationId`が正しいかを確認することもできますが、ここに示す簡単な方法は、`db.update`の`where`入力に`organizationId`を追加することです。この更新呼び出しは、organizationIdが一致しない場合に失敗します。

```ts
import { resolver } from "@blitzjs/rpc"
import db from "db"
import * as z from "zod"

const UpdateProject = z
  .object({
    id: z.number(),
    name: z.string(),
  })
  .nonstrict()

export default resolver.pipe(
  resolver.zod(UpdateProject),
  resolver.authorize(),
  async ({ id, ...data }, ctx) => {
    if (!ctx.session.orgId) throw new Error("Missing session.orgId")

    // Looks for both `id` and `organizationId`
    const project = await db.project.findFirst({
      where: { id, organizationId: ctx.session.orgId },
    })
    if (!project) throw new NotFoundError()

    const project = await db.project.update({ where: { id }, data })
    return project
  }
)
```

## 高度な認証 {#authorization}

if/else内で`ctx.session.$authorize()`を呼び出すような高度な認証も行えます

```ts
import { resolver } from "@blitzjs/rpc"
import db, { GlobalRole, MembershipRole } from "db"
import * as z from "zod"

const UpdateProject = z
  .object({
    id: z.number(),
    organizationId: z.number(),
    name: z.string(),
  })
  .nonstrict()

export default resolver.pipe(
  resolver.zod(UpdateProject),
  // Ensure all users are logged in
  resolver.authorize(),
  async ({ id, organizationId, ...data }, ctx) => {
    // if organizationId doesn't match current organization
    if (organizationId !== ctx.session.orgId) {
      // Require SUPERADMIN role
      ctx.session.$authorize(GlobalRole.SUPERADMIN)
    } else if (!ctx.session.accessibleProjects.includes(id)) {
      // If user doesn't have specific access to this project,
      // require them to be a project manager
      ctx.session.$authorize(MembershipRole.PROJECT_MANAGER)
    }

    // Looks for both `id` and `organizationId`
    const project = await db.project.findFirst({
      where: { id, organizationId },
    })
    if (!project) throw new NotFoundError()

    const project = await db.project.update({ where: { id }, data })

    return project
  }
)
```

#### ユーティリティ

ここに、SUPERADMINがすべての組織にアクセスできるようにしながら、通常のユーザーが現在ログインしている組織にのみアクセスできるようにする、高度な認可を可能にするユーティリティを示します。

```ts
// src/orders/queries/getOrder.ts
import { NotFoundError } from "blitz"
import { resolver } from "@blitzjs/rpc"
import db from "db"
import * as z from "zod"
import {
  enforceAdminOrProctorIfNotCurrentOrganization,
  setDefaultOrganizationId,
} from "src/core/utils"

const GetOrder = z.object({
  id: z.number(),
  organizationId: z.number().optional(),
})

export default resolver.pipe(
  resolver.zod(GetOrder),
  // Ensure user is logged in
  resolver.authorize(), //highlight-line
  // Set input.organizationId to the current organization if one is not set
  // This allows SUPERADMINs to pass in a specific organizationId
  setDefaultOrganizationId, //highlight-line
  // But now we need to enforce input.organizationId matches
  // session.orgId unless user is a SUPERADMIN
  enforceSuperAdminIfNotCurrentOrganization, //highlight-line
  async ({ id, organizationId }) => {
    const order = await db.getOrder({
      where: {
        id,
        // Now we can safely use organizationId to filter queries
        organizationId,
      },
    })
    if (!order) throw new NotFoundError()
    return order
  }
)
```

```ts
// src/core/utils.ts
import { Ctx } from "blitz"
import { Prisma, GlobalRole } from "db"

export default function assert(
  condition: any,
  message: string
): asserts condition {
  if (!condition) throw new Error(message)
}

export const setDefaultOrganizationId = <T extends Record<any, any>>(
  input: T,
  { session }: Ctx
): T & { organizationId: Prisma.IntNullableFilter | number } => {
  assert(
    session.orgId,
    "Missing session.orgId in setDefaultOrganizationId"
  )
  if (input.organizationId) {
    // Pass through the input
    return input as T & { organizationId: number }
  } else if (session.roles?.includes(GlobalRole.SUPERADMIN)) {
    // Allow viewing any organization
    return { ...input, organizationId: { not: 0 } }
  } else {
    // Set organizationId to session.orgId
    return { ...input, organizationId: session.orgId }
  }
}

export const enforceSuperAdminIfNotCurrentOrganization = <
  T extends Record<any, any>
>(
  input: T,
  ctx: Ctx
): T => {
  assert(ctx.session.orgId, "missing session.orgId")
  assert(input.organizationId, "missing input.organizationId")

  if (input.organizationId !== ctx.session.orgId) {
    ctx.session.$authorize(GlobalRole.SUPERADMIN)
  }
  return input
}
```

さらに高度な認証を行いたい場合は、[Blitz Guard](https://github.com/ntgussoni/blitz-guard)をチェックしてください。
```