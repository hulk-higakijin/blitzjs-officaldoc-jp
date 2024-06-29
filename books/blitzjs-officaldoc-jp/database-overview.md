---
title: データベース概要
---

:::message
あなたがデータベースに全く新しいのであれば、[Prismaのデータガイド](https://www.prisma.io/dataguide/)をチェックしてください。ここには、知りたいと思うことの大部分が網羅されています。
:::

デフォルトでは、Blitzは強く型付けされたデータベースクライアントである[Prisma](https://prisma.io)を使用します。

> PrismaはBlitzの必須要件ではありません。使いたいものは何でも使用できます。例えば、Mongo、TypeORMなど。

##### [Prismaのドキュメントをここで読む](https://www.prisma.io/docs/understand-prisma/introduction)

## データベーステーブルを追加する {#add-a-database-table}

1. `db/schema.prisma`を開き、次のようなモデルを追加します：

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

> 必要に応じて、
> [Prisma Schemaのドキュメントを参照してください](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/data-model)

2. その後、変更を適用するためにターミナルで`blitz prisma migrate dev`を実行します
3. これで、`db`を`db/index.ts`からインポートし、次のようにモデルを作成できます：
   - `db.project.create({data: {name: 'Hello'}})`

## PostgreSQLに切り替える {#switch-to-postgre-sql}

デフォルトでは、BlitzアプリはローカルのSQLiteデータベースで作成されます。代わりにPostgreSQLを使用したい場合は、次のステップを実行する必要があります：

1. [Postgresをローカルにインストール、実行する](./postgres)。
1. `db/schema.prisma`を開き、dbプロバイダーの値を`"sqlite"`から`"postgres"`に次のように変更します：

```prisma
datasource db {
  provider = "postgres"
  url      = env("DATABASE_URL")
}
```

2. `.env.local`で`DATABASE_URL`を変更します。便宜上、既にコメントアウトされたPostgreSQLの`DATABASE_URL`が存在します。セットアップに応じて、URLを修正する必要があります。
3. テスト実行時にもPostgreSQLを使用するように`.env.test.local`を更新します（`.env.local`とは異なるデータベースを指す必要があります）。
4. `db/migrations`フォルダーを削除します。
5. `blitz prisma migrate dev`を実行します。このコマンドは、データベースがまだ存在しない場合はデータベースを作成し、スキーマに基づいてテーブルを作成します。
