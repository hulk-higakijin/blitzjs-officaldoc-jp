---
title: クエリリゾルバ
sidebar_label: クエリリゾルバ
---

Blitzクエリはプレーンな非同期JavaScript関数であり、常にサーバー上で実行されます。

**1. クエリは必ず`queries`フォルダ内に配置する必要があります。以下の例はどれも有効です:**

- `src/queries/getProject.ts`
- `src/projects/queries/getProject.ts`
- `src/admin/projects/queries/getProject.ts`

**2. クエリは関数をデフォルトエクスポートとしてエクスポートする必要があります**

ここでは、データベースへのアクセスやサードパーティAPIからの取得など、通常のNode.jsコードを記述できます。

#### 引数

- `input: any`
  - 最初の引数はクエリに必要なものは何でも使えます。特別な要件はありません。
- `ctx: Ctx`
  - このコンテキストオブジェクトはBlitzによって実行時に自動的に提供されます。[HTTPミドルウェア](./middleware)により、データや関数をクエリに提供するために使用されます。例えば、認証ミドルウェアは`ctx.session`を提供し、現在のユーザーセッションに関するすべてのデータを含むことができます。

#### クエリの例

```ts
// src/products/queries/getProduct.tsx
import { Ctx } from "blitz"
import db from "db"
import * as z from "zod"

const GetProject = z.object({
  id: z.number(),
})

export default async function getProject(
  input: z.infer<typeof GetProject>,
  ctx: Ctx
) {
  // 入力を検証する
  const data = GetProject.parse(input)

  // ユーザーがログインしていることを要求する
  ctx.session.$authorize()

  const project = await db.project.findOne({ where: { id: data.id } })

  // 任意の処理や他のAPIからの取得を行うことができます

  return project
}
```

_プロジェクトのルートを自動的にエイリアス化しているため、`import db from 'db'`は`<project_root>/db/index.ts`をインポートしています_

次に、これらのクエリをあなたのコンポーネントでどのように使用するかについては[こちらのドキュメント](./query-usage)をお読みください。
