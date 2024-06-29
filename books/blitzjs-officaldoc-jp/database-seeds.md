```mdx
---
title: データベースシード
sidebar_label: シード
---

## について {#about}

デフォルトでは、Blitzにはテストデータを追加して
環境を素早くシードするための`db/seeds.ts`があります。

これは、DBを壊した後に再構築するのに役立ちますし、単に古いテストデータでいっぱいのときにも有用です。良いシードは、新しい人がプロジェクトに慣れるのをよりフレンドリーにしてくれることもあります（もう、このページに行って新しいユーザーを作成し、それから彼らのためにいくつかのタスクを作成してくださいと言う必要はありませんが、少なくとも1つのプロジェクトを作成することを忘れないでください...）。唯一の要件は、このファイルがデフォルトエクスポートを行うことです。

## 例 {#example}

0. これがあなたの例の`db/schema.prisma`だと仮定します：

```prisma
model Project {
  id        Int      @default(autoincrement()) @id
  name      String
  tasks     Task[]
}

model Task {
  id          Int      @default(autoincrement()) @id
  name        String
  project     Project  @relation(fields: [projectId], references: [id])
  projectId   Int
}
```

1. `db/seeds.ts`を追加します

```ts
import db from "./index"

const seed = async () => {
  const project = await db.project.create({ data: { name: "FooBar" } })

  for (let i = 0; i < 5; i++) {
    await db.task.create({
      data: {
        name: `Task: ${i}`,
        project: {
          connect: {
            id: project.id,
          },
        },
      },
    })
  }
}

export default seed
```

2. さて、シンプルに`blitz db seed`を実行するだけで完了です。データベースに新しいデータがすべて揃っているはずです。

> ヒント： [chance](https://chancejs.com/) や [faker](https://fakerjs.dev/) のようなライブラリを使用して、より現実的なデータ（名前、住所、メール、場所、日付、さらにはロレムイプサム）を持つようにしましょう
```