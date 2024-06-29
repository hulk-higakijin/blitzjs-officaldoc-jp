---
title: blitz generate
---

**エイリアス: `blitz g`**

このコマンドを使用して、プロジェクトの退屈なコードをスキャフォールドすることができます。

ページ、クエリ、ミューテーション、および Prisma モデルを生成できます。組み込みテンプレートに基づくカスタムテンプレートのサポートが間もなく登場予定であり、ジェネレーターをアプリのニーズに合わせてカスタマイズできます。

```bash
blitz generate [type] [model] [fields]
```
| 引数      | 必須    | 説明                                            |
| --------- | ------- | ------------------------------------------------|
| `type`    | はい    | 生成するファイルのタイプ。オプションは以下に示されています。 |
| `model`   | はい    | ファイルを生成するモデル名 \*                     |
| `fields`  | いいえ  | 生成するモデルのフィールドのリスト。[以下を参照](#model-fields) |

> \* `model` は、以下の予約語やその複数形ではあってはなりません: `page`, `api`, `query` または `mutation`。

以下は、どのコマンドがどのファイルを生成するかのマトリックスです。

| タイプ         | モデル | クエリ    | ミューテーション | ページ |
| ------------- | ----- | -------- | ------------- | ----- |
| `all`         | はい  | はい      | はい           | はい  |
| `resource`    | はい  | はい      | はい           |       |
| `model`       | はい  |          |               |       |
| `crud`        |       | はい      | はい           |       |
| `queries`     |       | はい      |               |       |
| `query`       |       | はい      |               |       |
| `mutations`   |       |          | はい           |       |
| `mutation`    |       |          | はい           |       |
| `pages`       |       |          |               | はい  |

##### 出力例

`blitz generate all project` は以下のファイルを生成します:

```
app/pages/projects/[projectId]/edit.tsx
app/pages/projects/[projectId].tsx
app/pages/projects/index.tsx
app/pages/projects/new.tsx
app/projects/components/ProjectForm.tsx
app/projects/queries/getProject.ts
app/projects/queries/getProjects.ts
app/projects/mutations/createProject.ts
app/projects/mutations/deleteProject.ts
app/projects/mutations/updateProject.ts
```

上記の例では、生成されたプロジェクトのインデックスページを以下で表示できます:
[localhost:3000/projects](http://localhost:3000/projects)

## オプション {#options}

##### `context/model`

プロジェクト内のファイルの整理のために、生成するファイルのネストされたフォルダパスを指定できます。

```bash
blitz generate all admin/products

// ファイルを `app/products` の代わりに `app/admin/products` に生成します
```

または、`--context` または `-c` オプションを使用してフォルダパスを指定することもできます。

##### `--parent`

省略形: `-p`

親モデルの子としてファイルを生成したい場合に使用します。

例えば、`Project` と `Task` モデルがあるとしましょう。`Task` は `Project` に属し、`Project` は多くの `Task` を持ちます。次のコマンドを実行します:

```bash
blitz generate all task --parent project
```

これにより、以下のファイルが生成されます:

```
app/pages/projects/[projectId]/tasks/[taskId]/edit.tsx
app/pages/projects/[projectId]/tasks/[taskId].tsx
app/pages/projects/[projectId]/tasks/index.tsx
app/pages/projects/[projectId]/tasks/new.tsx
app/tasks/components/TaskForm.tsx
app/tasks/queries/getTask.ts
app/tasks/queries/getTasks.ts
app/tasks/mutations/createTask.ts
app/tasks/mutations/deleteTask.ts
app/tasks/mutations/updateTask.ts
```

注意点として、これはモデル間の関係を生成しないことです。生成されるのはクエリ、ミューテーション、およびページのみです。

##### `--dry-run`

省略形: `-d`

生成されるファイルを表示しますが、ディスクに書き込みません。

#### `--env`

省略形: `-e`

アプリの環境名を設定することができます。
[詳細はこちら](/docs/custom-environments#custom-environments)をご覧ください。

### 基本的な例 {#basic-examples}

```bash
blitz generate all project
```

```bash
blitz generate mutations project
```

```bash
blitz generate crud admin/topsecret/files
```

```bash
blitz generate pages tasks --parent=projects
```

## モデル生成 {#model-generation}

以下のすべてのコマンドは、Prisma スキーマファイルにモデルを生成します:

- `blitz generate all`
- `blitz generate resource`
- `blitz generate model`

### モデルフィールド {#model-fields}

モデルフィールドは以下のように追加できます:

```bash
blitz generate model [fieldName]:[type]:[attribute]
```

- `fieldName`: データベース列の名前。任意のものを使用可能
  - 関係モデルを追加するには `belongsTo` を使用します。例: `belongsTo:user`
- `type`:
  - **オプション** - 指定されていない場合は `string` がデフォルト
  - **値:** `string`, `boolean`, `int`, 'bigint', `float`, `dateTime`, `json`, `uuid`, またはモデル名
  - `foo:uuid` は `foo:string:default=uuid` の省略形
  - `?` を追加して型をオプションにする: `string?`

    **注意:** ターミナルで `zsh` を使用している場合、フィールドを正しく解釈させるには引用符（`""`）で囲む必要があります。例: `"name:string?"`

  - `[]` を追加して型をリストにする: `task[]`

- `attribute`:
  [prisma フィールド属性](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/data-model#attributes) を追加するためのもの
  - **オプション**
  - サポートされているもの: `default`, `unique`
  - 属性に引数が必要な場合は `=` を使用して指定できます。例: `default=false`。これにより、デフォルト値が `false` に設定されます。

詳細は以下を参照してください:
[Prisma スカラ型のドキュメント](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference/#model-field-scalar-types)
または
[Prisma リレーションのドキュメント](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/relations)。

##### 例

```bash
> blitz generate all project
> blitz generate all project name
> blitz generate all project name:string description:string? active:boolean
```

##### スカラフィールド

```bash
> blitz generate model puppy isCute:boolean
> blitz generate model rocket launchedAt:dateTime
> blitz generate model task completed:boolean:default=false
```

##### Has One リレーション

```bash
blitz g model project task:Task
```

これにより、次のように生成されます:

```prisma
model Project {
  ...
  task     Task
}
```

##### Has Many リレーション

```bash
blitz g model project tasks:Task[]
```

これにより、次のように生成されます:

```prisma
model Project {
  ...
  tasks     Task[]
}
```

##### Belongs To リレーション

```bash
blitz g model task belongsTo:project
```

これにより、次のように生成されます:

```prisma
model Task {
  ...
  project   Project? @relation(fields: [projectId], references: [id])
  projectId Int?
}
```

##### 完全な例

```bash
blitz generate model task \
       name \
       completed:boolean:default=false \
       belongsTo:project?
```

これにより、次のように生成されます:

```prisma
model Task {
  id        Int      @default(autoincrement()) @id
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  name      String
  completed Boolean  @default(false)
  project   Project? @relation(fields: [projectId], references: [id])
  projectId Int?
}
```

### モデルの更新 {#updating-a-model}

`blitz generate model` コマンドを再度実行すると、既存のモデルにフィールドが追加されます。以下のコマンドは `Task` モデルに `subheading` フィールドを追加します。

```bash
blitz generate model task subheading:string
```

### カスタムテンプレート {#custom-templates}

デフォルトのテンプレートではなく、`blitz generate` でカスタムテンプレートを使用したい場合（例: 異なるスタイルで）、`src/blitz-server` ファイルで `cliConfig` オブジェクトをエクスポートすることで指定できます。

```ts
import type { BlitzCliConfig } from "blitz"
...
export const cliConfig: BlitzCliConfig = {
  customTemplates: "src/templates",
}
```

#### カスタムテンプレートの生成

```bash
blitz generate custom-templates
```

上記のコマンドは、使用するための出発点として blitz が使用するデフォルトのテンプレートを指定したディレクトリに生成します。また、`src/blitz-server` ファイルの `cliConfig` 設定を最新のデータで更新し、即時使用を可能にします。

### コード生成 {#codegen}

[`blitz generate`](/docs/cli-generate)コマンドを使用して、以下のコードを生成できます:
- Prisma モデル
- ページ
- フォーム
- クエリ
- ミューテーション
モデルに基づいてコンテンツを生成するために、テンプレート内のフォームと `zod` 検証スキーマを使用します。新しい Blitz アプリケーションでは次の行を見ることができます:
```js
// template: __fieldName__: z.__zodType__(),
/* template: <__component__ name="__fieldName__" label="__Field_Name__" placeholder="__Field_Name__" /> */
```
テンプレートの値をコード生成設定に基づいて置き換えます。これがデフォルトの設定です:
```json
{
  "string": {
    "component": "LabeledTextField",
    "inputType": "text",
    "zodType": "string",
    "prismaType": "String"
  },
  "boolean": {
    "component": "LabeledTextField",
    "inputType": "text",
    "zodType": "boolean",
    "prismaType": "Boolean"
  },
  "int": {
    "component": "LabeledTextField",
    "inputType": "number",
    "zodType": "number",
    "prismaType": "Int"
  },
  "number": {
    "component": "LabeledTextField",
    "inputType": "number",
    "zodType": "number",
    "prismaType": "Int"
  },
  "bigint": {
    "component": "LabeledTextField",
    "inputType": "number",
    "zodType": "number",
    "prismaType": "BigInt"
  },
  "float": {
    "component": "LabeledTextField",
    "inputType": "number",
    "zodType": "number",
    "prismaType": "Float"
  },
  "decimal": {
    "component": "LabeledTextField",
    "inputType": "number",
    "zodType": "number",
    "prismaType": "Decimal"
  },
  "datetime": {
    "component": "LabeledTextField",
    "inputType": "text",
    "zodType": "string",
    "prismaType": "DateTime"
  },
  "uuid": {
    "component": "LabeledTextField",
    "inputType": "text",
    "zodType": "string().uuid",
    "prismaType": "String",
    "default": "uuid"
  },
  "json": {
    "component": "LabeledTextField",
    "inputType": "text",
    "zodType": "any",
    "prismaType": "Json"
  }
}
```
新しいタイプを追加したり、既存のタイプをオーバーライドしたりすることができます。`component`, `inputType`, `zodType`, `prismaType` フィールドは必須で、`default` フィールドはオプションです — これは Prisma モデル定義のデフォルト値として使用されます。テンプレートを拡張したい場合は、フィールドの設定内に新しいキーを提供できます。
例 `src/blitz-server.ts`:
```ts
import type { BlitzCliConfig } from "blitz"
...
export const cliConfig: BlitzCliConfig = {
  codegen: {
    fieldTypeMap: {
      string: {
        component: "LabeledTextField",
        inputType: "text",
        zodType: "string",
        prismaType: "String",
      },
      boolean: {
        component: "LabeledTextField",
        inputType: "text",
        zodType: "boolean",
        prismaType: "Boolean",
      },
      // ...
    },
  },
}
```
