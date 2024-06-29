```mdx
---
title: Mutation Resolvers
sidebar_label: Mutation Resolvers
---

Blitzのミューテーションはプレーンな非同期JavaScript関数で、常にサーバー側で実行されます。

**1. ミューテーションは`mutations`フォルダ内になければなりません。次のものすべてが有効です:**

- `src/mutations/createProject.ts`
- `src/projects/mutations/createProject.ts`
- `src/admin/projects/mutations/createProject.ts`

**2. ミューテーションはデフォルトエクスポートとして関数をエクスポートする必要があります**

ここでは、データベースアクセスやサードパーティのAPIからの取得を含む、通常のNode.jsコードを記述できます。

#### 引数

- `input: any`
  - 最初の引数はミューテーションに必要なものです。特別な要件はありません。
- `ctx: Ctx`
  - このコンテキストオブジェクトは実行時にBlitzによって自動的に提供されます。これは[HTTPミドルウェア](./middleware)によってデータや関数をミューテーションに提供するために使用されます。たとえば、認証ミドルウェアは現在のユーザーセッションに関するすべてのデータを持つ`ctx.session`を提供できます。

#### ミューテーションの例

```ts
// app/products/mutations/createProduct.tsx
import { Ctx } from "blitz"
import db from "db"
import * as z from "zod"

const CreateProject = z
  .object({
    name: z.string(),
  })
  .nonstrict()

export default async function createProject(
  input: z.infer<typeof CreateProject>,
  ctx: Ctx
) {
  // 入力を検証 - セキュリティのために非常に重要
  const data = CreateProject.parse(input)

  // ユーザーがログインしていることを要求
  ctx.session.$authorize()

  const project = await db.project.create({ data })

  // 他のAPIからの取得など、任意の処理を実行可能

  return project
}
```

_プロジェクトのルートを自動的にエイリアスするため、`import db from 'db'`は`<project_root>/db/index.ts`をインポートしています_

次に、これらのミューテーションをコンポーネントでどのように使用するかを知るために、[これらのドキュメントを読む](./mutation-usage)と良いでしょう。
```