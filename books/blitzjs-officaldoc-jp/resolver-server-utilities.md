---
title: サーバー ユーティリティ
sidebar_label: サーバー ユーティリティ
---

Blitzは、いくつかのユーティリティを含む `resolver` オブジェクトをエクスポートします。
ここで使用される "Resolver" および [queries](./query-resolvers) や [mutations](./mutation-resolvers) の用語は、入力を受け取り、それを出力または副作用に "解決" する関数を指します。この用語はGraphQLで一般的に使用されます。リゾルバーはAPIエンドポイントとは異なります。APIエンドポイントではNodeのリクエストおよびレスポンスオブジェクトへの完全なアクセスがあり、HTTP関連の管理を担当します。しかし、リゾルバーは生のAPIハンドラーから一段階取り除かれています。

以下のユーティリティは、一般的に一緒に使用されるため `resolverPipe` や `resolverZod` の代わりに `resolver` オブジェクトの下でエクスポートされます。これによりインポートが整理され、`resolver` オブジェクト上の他のユーティリティを簡単に見つけることができます。

## `resolver.pipe` {#resolver-pipe}

これは複雑なリゾルバーをより簡単かつクリーンに記述するための関数型パイプです。

パイプは、自動的にある関数の出力を次の関数に渡します。以下はBlitzを使用しない例です：

```ts
// パイプなし
export function (input1) {
  const output1 = function1(input1)
  const output2 = function2(output1)
  const output3 = function3(output2)
  return output3
}

// パイプ使用
export pipe(
  function1,
  function2,
  function3,
)
```

**利点:**

- 明示的なTypeScript型の必要性を減らします
- zodスキーマを使用して一行で入力データを検証できます（`resolver.zod()`）
- 一行でユーザーを認証できます（`resolver.authorize()`）
- 関数型合成を可能にします

### 例 {#example}

```ts
import { resolver } from "@blitzjs/rpc"
import db from "db"
import * as z from "zod"

export const CreateProject = z.object({
  name: z.string(),
  dueDate: z.date().optional(),
  orgId: z.number().optional(),
})

export default resolver.pipe(
  resolver.zod(CreateProject),
  resolver.authorize(),
  // デフォルトのorgIdを設定
  (input, { session }) => ({
    ...input,
    orgId: input.orgId ?? session.orgId,
  }),
  async (input, ctx) => {
    console.log("Creating project...", input.orgId)
    const project = await db.project.create({
      data: input,
    })
    console.log("Created project")
    return project
  }
)
```

<Card type="info">

パイプ全体のリゾルバー関数の入力型は、パイプの**最初**の引数の入力型によって決まります。

このため、ほとんどの場合、`resolver.zod()`をパイプの最初に置く必要があります。

</Card>

### API {#api}

#### 引数

1つからN個の関数を受け取り、入力データを最初の引数として取り、[ミドルウェア `ctx`](./middleware) を第二引数として使用します。

最初の関数の出力は次の関数の最初の入力引数です。最後の関数の出力は、クライアントに送信される最終的なリゾルバーの結果データです。

TypeScript型は関数から次の関数へと自動的に推論されます。

```js
// 1つの関数
const resolver = resolver.pipe((input: ResolverInput, ctx: Ctx) => ResolverOutput)
// 2つの関数
const resolver = resolver.pipe(
  (input: ResolverInput, ctx: Ctx) => A),
  (input: A, ctx: Ctx) => ResolverOutput),
)
// 3つの関数
const resolver = resolver.pipe(
  (input: ResolverInput, ctx: Ctx) => A),
  (input: A, ctx: Ctx) => B),
  (input: B, ctx: Ctx) => ResolverOutput),
)
// ... など
```

#### 戻り値

これは、標準のリゾルバー型である `(input, ctx) => Promise<result>` の合成された関数を返します。

## `resolver.zod` {#resolver-zod}

これは優れた入力検証ライブラリである
[Zod](https://github.com/colinhacks/zod) を使用するための便利なユーティリティです。zodスキーマを受け取り、入力データに対して `schema.parseAsync` を実行します。

> **注意**: 同期バージョン（`schema.parse`）を実行したい場合は、resolver.zodの第二引数に文字列 "sync" を渡すことができます
> 例：`resolver.zod(CreateProject, "sync")`

これを使用する必要はありません。メインのリゾルバー関数内で `const safeInput = CreateProject.parse(input)` を直接追加できます。ただし、その場合は関数インターフェースを手動で型付けする必要があります。このユーティリティは、入力型を設定し、実行時に入力データを検証する一方で、これを一度に行います。

### 例 {#example-1}

```ts
import { resolver } from "@blitzjs/rpc"
import * as z from "zod"

export const CreateProject = z.object({
  name: z.string(),
  dueDate: z.date().optional(),
  orgId: z.number().optional(),
})

export default resolver.pipe(
  resolver.zod(CreateProject), // ハイライト行
  async (input, ctx) => {
    // 処理
  }
)
```

### API {#api-1}

```js
const pipeFn = resolver.zod(MyZodSchema)
```

#### 引数

- `ZodSchema:` [zod](https://github.com/colinhacks/zod) スキーマ
  - **必須**

#### 戻り値

`resolver.pipe` に与える関数
`(rawInput, ctx: Ctx) => validInput`

## `resolver.authorize` {#resolver-authorize}

`resolver.pipe` で `resolver.authorize` を使用することは、ユーザーがクエリまたはミューテーションを呼び出す権限があるかどうかを簡単に確認する方法です。Blitzはコンテキストオブジェクトから
[`session.$authorize`](./authorization#isauthorized-adapters) を使用します。

### 例 {#example-2}

```ts
import { resolver } from "@blitzjs/rpc"

export default resolver.pipe(
  // ハイライトスタート
  resolver.authorize(),
  // resolver.authorize('admin'),
  // ハイライトエンド
  async (input, ctx) => {
    // 処理
  }
)
```

### API {#api-2}

```js
const pipeFn = resolver.authorize(...isAuthorizedArgs)
```

#### 引数

- `...isAuthorizedArgs`: あなたの
  [`isAuthorized` アダプター](./authorization#isauthorized-adapters) と同じ引数

#### 戻り値

`resolver.pipe` に与える関数
型 `(input, ctx: Ctx) => input`

## `paginate` {#paginate}

これはクエリのページネーションのための便利なユーティリティです

### 例 {#example-3}

```ts
import { paginate, Ctx } from "@blitzjs/rpc"
import db, { Prisma } from "db"

interface GetProjectSInput
  extends Pick<
    Prisma.ProjectFindManyArgs,
    "where" | "orderBy" | "skip" | "take"
  > {}

export default async function getProjects(
  { where, orderBy, skip = 0, take = 100 }: GetProjectsInput,
  ctx: Ctx
) {
  ctx.session.$authorize()

  const {
    items: projects,
    hasMore,
    nextPage,
    count,
  } = await paginate({
    count: () => db.project.count({ where }),
    query: (paginateArgs) =>
      db.project.findMany({ ...paginateArgs, where, orderBy }),
    skip,
    take,
  })

  return {
    projects,
    nextPage,
    hasMore,
    count,
  }
}
```

### API {#api-3}

```js
const paginationData = await paginate(paginateArguments)
```

#### 引数

- `count`: `() => Promise <number>`
  - **必須**
  - 数値を解決するプロミスを返す関数
- `query`: `(payload: { skip: number; take: number }) => Promise<T>`
  - **必須**
  - ページネーションのペイロードを受け取り、何かを返す関数（戻り値の型はページネーション結果に継承されます）。
- `skip`: `number`
  - スキップする行数
- `take`: `number`
  - 取得する行数
- `maxTake`: `number`
  - `take` の最大値。これは、ユーザーが送信する "take" を保護します。
  - デフォルトは250。

#### 戻り値

- `paginationData`
  - `items`: `T` （`query` 関数により解決されたデータ）
  - `nextPage`: `{ skip: number, take: number } | null` （次のページのページネーションペイロード）
  - `hasMore`: `boolean` （ロードする項目がまだあるかどうか。）
  - `count`: `number` （総項目数。`count` 関数により解決されます）
