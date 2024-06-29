---
title: チュートリアル
---

このチュートリアルでは、基本的な投票アプリケーションの作成方法を説明します。

既に [Blitz をインストール](./get-started) していることを前提とします。Blitz がインストールされているかどうか、およびインストールされているバージョンを確認するには、ターミナルで次のコマンドを実行します：

```sh
blitz -v
```

Blitz がインストールされている場合、インストールされているバージョンが表示されます。インストールされていない場合、「command not found: blitz」などのエラーが表示されます。

## 新しいアプリの作成 {#creating-a-new-app}

コマンドラインで、アプリを作成したいフォルダに `cd` してから、次のコマンドを実行します：

```sh
blitz new my-blitz-app
```

Blitz は現在のフォルダに `my-blitz-app` フォルダを作成します。新しいアプリの設定方法を尋ねられます。このチュートリアルでは、すべてのデフォルト値を選択するため、**Enter** キーを押すだけで進みます（TypeScript、npm、React Final Form を使用した完全な Blitz アプリが作成されます）。

`blitz new` が作成した内容を見てみましょう：

```
my-blitz-app
├── src/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── SignupForm.tsx
│   │   ├── mutations/
│   │   │   ├── changePassword.ts
│   │   │   ├── forgotPassword.test.ts
│   │   │   ├── forgotPassword.ts
│   │   │   ├── login.ts
│   │   │   ├── logout.ts
│   │   │   ├── resetPassword.test.ts
│   │   │   ├── resetPassword.ts
│   │   │   └── signup.ts
│   │   └── validations.ts
│   ├── core/
│   │   ├── components/
│   │   │   ├── Form.tsx
│   │   │   └── LabeledTextField.tsx
│   │   └── layouts/
│   │       └── Layout.tsx
│   ├── users/
│   │   ├── hooks/
│   │   │   └── useCurrentUser.ts
│   │   └── queries/
│   │       └── getCurrentUser.ts
│   ├── pages/
│   │   ├── api/
│   │   │   └── rpc/
│   │   │       └── [[...blitz]].ts
│   │   ├── auth/
│   │   │   ├── forgot-password.tsx
│   │   │   ├── login.tsx
│   │   │   └── signup.tsx
│   │   ├── _app.tsx
│   │   ├── _document.tsx
│   │   ├── 404.tsx
│   │   └── index.tsx
│   ├── blitz-client.ts
│   └── blitz-server.ts
├── db/
│   ├── migrations/
│   ├── index.ts
│   ├── schema.prisma
│   └── seeds.ts
├── integrations/
├── mailers/
│   └── forgotPasswordMailer.ts
├── public/
│   ├── favicon.ico*
│   └── logo.png
├── test/
│   └── setup.ts
├── README.md
├── next.config.js
├── vitest.config.ts
├── package.json
├── tsconfig.json
├── types.d.ts
├── types.ts
└── yarn.lock
```

これらのファイルは以下のようなものです：

- `src/` フォルダには Blitz のセットアップファイル — `blitz-client.ts` と `blitz-server.ts` が含まれています。また、ここにクエリ/ミューテーションやコンポーネントの一部を配置します。

- `src/pages/` フォルダは主なページフォルダです。ここにすべてのページと API ルートを配置します。

- `src/core/` フォルダは、アプリ全体で使用されるコンポーネント、フックなどを配置する主要な場所です。

- `db/` にはデータベースの設定が含まれます。モデルを書いたり、マイグレーションを確認したりする場合はここに移動します。

- `public/` フォルダには、静的アセットを配置します。アプリで使用する画像、ファイル、ビデオなどがある場合はここに配置します。

- `.npmrc`、`.env` などの「ドットファイル」は、さまざまな JavaScript ツールの設定ファイルです。

- `next.config.js` は Blitz と Next.js の高度なカスタム設定用です。

- `tsconfig.json` は TypeScript の推奨設定です。

[ファイル構造についての詳細はこちら](./file-structure) を参照してください。

## 開発サーバー {#the-development-server}

まだ `my-blitz-app` フォルダにいない場合はそこに移動して、次のコマンドを実行します：

```sh
blitz dev
```

ターミナルには次のような出力が表示されます：

```sh
✔ Compiled
Loaded env from /private/tmp/my-blitz-app/.env
warn  - You have enabled experimental feature(s).
warn  - Experimental features are not covered by semver, and may cause unexpected or broken application behavior. Use them at your own risk.

ready - started server on 0.0.0.0:3000, url: http://localhost:3000
event - compiled successfully
```

サーバーが起動したので、ウェブブラウザで [localhost:3000](http://localhost:3000) にアクセスします。Blitz のロゴが表示されたウェルカムページが表示されます。成功です！

## ユーザーとしてサインアップ {#sign-up-as-a-user}

Blitz アプリには、ユーザーのサインアップとログインが既にセットアップされています！さっそく試してみましょう。**サインアップ** ボタンをクリックし、任意のメールアドレスとパスワードを入力して **アカウントを作成** をクリックします。その後、ホームページにリダイレクトされ、ユーザーの `id` と `role` が表示されます。

ログアウトして再度ログインしてみることもできます。または、ログインページで **パスワードを忘れましたか？** をクリックして、そのフローを試してみてください。

## 最初のページを書いてみましょう {#write-your-first-page}

次に、最初のページを作成してみましょう。

`pages/index.tsx` ファイルを開き、`Home` コンポーネントの内容を次のように置き換えます：

```tsx
//...

const Home: BlitzPage = () => {
  return (
    <div>
      <h1>Hello, world!</h1>

      <Suspense fallback="Loading...">
        <UserInfo />
      </Suspense>
    </div>
  )
}

//...
```

ファイルを保存すると、ブラウザにページが更新されるはずです。これを自由にカスタマイズできます。準備ができたら、次のセクションに進みましょう。

## データベースのセットアップ {#database-setup}

嬉しいことに、SQLite データベースは既にセットアップされています！ターミナルで `blitz prisma studio` を実行すると、データベース内のデータを表示するためのウェブインターフェースが開きます。

最初の本格的なプロジェクトを開始する際には、将来的にデータベースの変更が必要になる可能性を避けるために、よりスケーラブルなデータベース（PostgreSQL など）を使用することを検討してください。詳細は [データベースの概要](database-overview) を参照してください。今のところ、デフォルトの SQLite データベースを使用し続けます。

## モデルのためのコードの自動生成 {#scaffolding-code-for-our-models}

Blitz はボイラープレートコードを自動生成する便利な CLI コマンド [`generate`](./cli-generate) を提供しています。`generate` を使用して、`Question` と `

Choice` の2つのモデルを作成します。`Question` には質問のテキストと選択肢のリストがあります。`Choice` には選択肢のテキスト、投票数、および関連する質問があります。Blitz は両方のモデルに自動的に id、作成タイムスタンプ、および最終更新タイムスタンプを生成します。

#### まず、`Question` モデルに関連するすべてを生成します：

```sh
blitz generate all question text:string
```

プロンプトが表示されたら、**Enter** を押して `prisma migrate` を実行し、新しいモデルでデータベーススキーマを更新します。名前を入力するように求められるので、「add question」などと入力します。

```
CREATE    src/pages/questions/[questionId].tsx
CREATE    src/pages/questions/[questionId]/edit.tsx
CREATE    src/pages/questions/index.tsx
CREATE    src/pages/questions/new.tsx
CREATE    src/questions/components/QuestionForm.tsx
CREATE    src/questions/queries/getQuestion.ts
CREATE    src/questions/queries/getQuestions.ts
CREATE    src/questions/mutations/createQuestion.ts
CREATE    src/questions/mutations/deleteQuestion.ts
CREATE    src/questions/mutations/updateQuestion.ts

✔ モデル 'Question' が schema.prisma に作成されました：

>
> model Question {
>   id        Int      @id @default(autoincrement())
>   createdAt DateTime @default(now())
>   updatedAt DateTime @updatedAt
>   text      String
> }
>

✔ データベースを更新するために 'prisma migrate dev' を実行しますか？ (Y/n) · true
環境変数が .env からロードされました
Prisma スキーマが db/schema.prisma からロードされました
データソース "db": SQLite データベース "db.sqlite" at "file:./db.sqlite"

✔ 新しいマイグレーションの名前を入力してください： … add question
次のマイグレーションが新しいスキーマの変更から作成および適用されました：

migrations/
  └─ 20210722070215_add_question/
    └─ migration.sql

データベースはスキーマと同期しました。

✔ 187ms で ./node_modules/@prisma/client に Prisma クライアント (4.0.0) を生成しました
```

`generate` コマンドで `all` タイプを指定すると、モデル、クエリ、ミューテーション、ページファイルが生成されます。利用可能なタイプオプションのリストについては、[Blitz generate](./cli-generate) ページを参照してください。

#### 次に、対応するクエリとミューテーションを持つ `Choice` モデルを生成します。

今回は `Choice` モデルのページを生成する必要がないため、`resource` タイプを指定します：

```sh
blitz generate resource choice text votes:int:default=0 belongsTo:question
```

エラーが発生した場合は `blitz prisma format` を実行します。

これはまだ `Question` モデルに `Choice` フィールドを追加していないため、データベースのマイグレーションを必要としません。したがって、プロンプトが表示されたらマイグレーションを実行しないように `false` を選択します：

```
CREATE    src/choices/queries/getChoice.ts
CREATE    src/choices/queries/getChoices.ts
CREATE    src/choices/mutations/createChoice.ts
CREATE    src/choices/mutations/deleteChoice.ts
CREATE    src/choices/mutations/updateChoice.ts

✔ モデル 'choice' が schema.prisma に作成されました：

> model Choice {
>   id         Int      @default(autoincrement()) @id
>   createdAt  DateTime @default(now())
>   updatedAt  DateTime @updatedAt
>   text       String
>   votes      Int      @default(0)
>   question   Question @relation(fields: [questionId], references: [id])
>   questionId Int
> }

? データベースを更新するために 'prisma migrate dev' を実行しますか？ (Y/n) › false
```

#### 最後に `Question` モデルを更新して `Choice` とのリレーションシップを持たせます。

`db/schema.prisma` を開き、`Question` モデルに `choices Choice[]` を追加します。

```diff
model Question {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  text      String
+ choices   Choice[]
}
```

次にデータベースを更新するためにマイグレーションを実行します：

```sh
blitz prisma migrate dev
```

再び、マイグレーションの名前を入力します。「add choice」など：

```
環境変数が .env からロードされました
Prisma スキーマが db/schema.prisma からロードされました
データソース "db": SQLite データベース "db.sqlite" at "file:./db.sqlite"

✔ マイグレーションの名前を入力してください… add choice
次のマイグレーションが新しいスキーマの変更から作成および適用されました：

migrations/
  └─ 20210412175528_add_choice/
    └─ migration.sql

データベースはスキーマと同期しました。
```

これでデータベースの準備が整い、Prisma クライアントも生成されました。次に進んで Prisma クライアントで遊んでみましょう！

## モデル属性のために生成されたコードの更新 {#update-generated-code-for-our-model-attributes}

:::note info
インフォメーション
アプリを再度実行する前に、生成されたコードの一部をカスタマイズする必要があります。最終的にはこれらの修正は不要になりますが、現在のところ、いくつかの未解決の問題を回避する必要があります。
:::

生成されたページの内容は、生成中に定義した実際のモデル属性を使用していません。これを修正するために、生成されたページを修正しましょう。

### 質問ページ {#question-pages}

[//]: #
"generate が実際のモデル属性を使用するようになったらこのセクションを削除してください"

`src/pages/questions/index.tsx` にジャンプします。生成された `QuestionsList` コンポーネントに注目してください：

```tsx
// src/pages/questions/index.tsx

export const QuestionsList = () => {
  const router = useRouter()
  const page = Number(router.query.page) || 0
  const [{ questions, hasMore }, { isPreviousData }] = usePaginatedQuery(
    getQuestions,
    {
      orderBy: { id: "asc" },
      skip: ITEMS_PER_PAGE * page,
      take: ITEMS_PER_PAGE,
    }
  )

  const goToPreviousPage = () =>
    router.push({ query: { page: page - 1 } })

  const goToNextPage = () => {
    if (!isPreviousData && hasMore) {
      router.push({ query: { page: page + 1 } })
    }
  }

  return (
    <div>
      <ul>
        {questions.map((question) => (
          <li key={question.id}>
            <Link
              href={Routes.ShowQuestionPage({ questionId: question.id })}
            >
              <a>{question.name}</a>
            </Link>
          </li>
        ))}
      </ul>

      <button disabled={page === 0} onClick={goToPreviousPage}>
        Previous
      </button>
      <button
        disabled={isPreviousData || !hasMore}
        onClick={goToNextPage}
      >
        Next
      </button>
    </div>
  )
}
```

これでは動作しません！上記で作成した `Question` モデルには `name` フィールドがありません。これを修正するために、`question.name` を `question.text` に置き換えます：

```diff
// src/pages/questions/index.tsx

export const QuestionsList = () => {
  const router = useRouter()
  const page = Number(router.query.page) || 0
  const [{questions, hasMore}, {isPreviousData}] = usePaginatedQuery(
    getQuestions, {
      orderBy: {id: "asc"},
      skip: ITEMS_PER_PAGE * page,
      take: ITEMS_PER_PAGE,
    },
  )

  const goToPreviousPage = () => router.push({query: {page: page - 1}})

  const goToNextPage = () => {
    if (!isPreviousData && hasMore) {
      router.push({query: {page: page + 1}})
    }
  }

  return (
    <div>
      <ul>
        {questions.map((question) => (
          <li key={question.id}>
            <Link href={Routes.ShowQuestionPage({ questionId: question.id })}>
-              <a>{question.name

}</a>
+              <a>{question.text}</a>
            </Link>
          </li>
        ))}
      </ul>

      <button disabled={page === 0} onClick={goToPreviousPage}>
        Previous
      </button>
      <button disabled={isPreviousData || !hasMore} onClick={goToNextPage}>
        Next
      </button>
    </div>
  )
}
```

次に、`src/questions/components/QuestionForm.tsx` に同様の修正を適用します。フォーム送信で、`LabeledTextField` の `name` を `"text"` に置き換えます。

```diff
export function QuestionForm<S extends z.ZodType<any, any>>(
  props: FormProps<S>,
) {
  return (
    <Form<S> {...props}>
-     <LabeledTextField name="name" label="Name" placeholder="Name" />
+     <LabeledTextField name="text" label="Text" placeholder="Text" />
    </Form>
  )
}
```

### `createQuestion` ミューテーション {#create-question-mutation}

`src/questions/mutations/createQuestion.ts` では、`CreateQuestion` の zod バリデーションスキーマを `name` から `text` に更新する必要があります。

```diff
// src/questions/mutations/createQuestion.ts

const CreateQuestion = z
  .object({
-   name: z.string(),
+   text: z.string(),
  })
// ...
```

### `updateQuestion` ミューテーション {#update-question-mutation}

`src/questions/mutations/updateQuestion.ts` では、`UpdateQuestion` の zod バリデーションスキーマを `name` から `text` に更新する必要があります。

```diff
// src/questions/mutations/updateQuestion.ts

const UpdateQuestion = z
  .object({
    id: z.number(),
-   name: z.string(),
+   text: z.string(),
  })
```

### `deleteQuestion` ミューテーション {#delete-question-mutation}

[//]: # "Prisma がカスケード削除をサポートしたらこのセクションを削除してください"

Prisma はまだ「カスケード削除」をサポートしていません。このチュートリアルの文脈では、質問を削除する際に `Choice` データが削除されないことを意味します。これを一時的に回避するために、生成された `deleteQuestion` ミューテーションを手動で修正します。テキストエディタで `src/questions/mutations/deleteQuestion.ts` を開き、関数本体の上部に次のコードを追加します：

```ts
await db.choice.deleteMany({ where: { questionId: id } })
```

最終結果は次のようになります：

```diff
// src/questions/mutations/deleteQuestion.ts

export default resolver.pipe(
  resolver.zod(DeleteQuestion),
  resolver.authorize(),
  async ({id}) => {
+   await db.choice.deleteMany({where: {questionId: id}})
    const question = await db.question.deleteMany({where: {id}})

    return question
  },
)
```

このミューテーションは、質問自体を削除する前に関連する選択肢を削除するようになります。

### `updateChoice` ミューテーション {#update-choice-mutation}

`src/choices/mutations/updateChoice.ts` では、`UpdateChoice` の zod バリデーションスキーマを `name` から `text` に更新する必要があります。

```diff
// src/choices/mutations/updateChoice.ts

const UpdateChoice = z
  .object({
    id: z.number(),
-   name: z.string(),
+   text: z.string(),
  })
```

## 不要なファイルの削除 {#remove-unnecessary-file}

スキャフォールディングにより、もはや必要ないミューテーションファイルが生成されました。`yarn tsc` または `git push` を成功させるためには、`src/choices/mutations/createChoice.ts`（未使用）を削除するか、CreateChoice zod スキーマを必要なフィールドで更新する必要があります。

[//]: # "blitz コンソールのサポートを追加した後にこのセクションを追加してください"

#### これで質問の作成、更新、および削除を試してみてください

素晴らしい！アプリケーションを停止し、ターミナルで `blitz dev` を再度実行し、`localhost:3000/questions` にアクセスして質問の作成、編集、削除を試してみてください。

## 質問フォームに選択肢を追加する {#adding-choices-to-the-question-form}

ここまで順調です！次に、質問フォームに選択肢を追加します。エディタで `src/questions/components/QuestionForm.tsx` を開きます。

3つの `<LabeledTextField>` コンポーネントを選択肢として追加します。

```diff
export function QuestionForm<S extends z.ZodType<any, any>>(
  props: FormProps<S>,
) {
  return (
    <Form<S> {...props}>
      <LabeledTextField name="text" label="Text" placeholder="Text" />
+     <LabeledTextField name="choices.0.text" label="Choice 1" />
+     <LabeledTextField name="choices.1.text" label="Choice 2" />
+     <LabeledTextField name="choices.2.text" label="Choice 3" />
    </Form>
  )
}
```

次に `src/questions/mutations/createQuestion.ts` を開き、選択肢のデータをミューテーションで受け入れるように zod スキーマを更新します。また、選択肢が作成されるように `db.question.create()` 呼び出しを更新します。その後、`QuestionForm` のバリデーションスキーマを作成するために次のステップで使用するために `CreateQuestion` zod スキーマをエクスポートする必要があります。

```diff
// src/questions/mutations/createQuestion.ts

+ export const CreateQuestion = z
    .object({
      text: z.string(),
+     choices: z.array(z.object({text: z.string()})),
    })

export default resolver.pipe(
  resolver.zod(CreateQuestion),
  resolver.authorize(),
  async (input) => {
-   const question = await db.question.create({data: input})
+   const question = await db.question.create({
+     data: {
+       ...input,
+       choices: {create: input.choices},
+     },
+   })

    return question
  },
)
```

次に、`QuestionForm` のバリデーションスキーマを格納するための別のファイルを作成します。`src/questions` フォルダに新しいファイル `validations.ts` を作成し、`CreateQuestion` 変数を `./mutations/createQuestion.ts` から新しい `validations.ts` ファイルに移動します。次に、`src/questions/mutations/createQuestion.ts` で `CreateQuestion` を `../validations` からインポートします。

```diff
// src/questions/validations.ts

+ import * as z from 'zod';

+ export const CreateQuestion = z.object({
+ 	text: z.string(),
+ 	choices: z.array(z.object({ text: z.string() }))
+ });
```

```diff
// src/questions/mutations/createQuestion.ts

import { resolver } from '@blitzjs/rpc';
import db from 'db';
- import { z } from 'zod';
+ import { CreateQuestion } from '../validations';

- const CreateQuestion = z.object({
- 	text: z.string(),
- 	choices: z.array(z.object({ text: z.string() }))
- });

export default resolver.pipe(resolver.zod(CreateQuestion), resolver.authorize(), async (input) => {
	// TODO: マルチテナントアプリの場合、正しいテナントを確保するためのバリデーションを追加する必要があります。
	const question = await db.question.create({
		data: {
			...input,
			choices: { create: input.choices }
		}
	});

	return question;
});
```

:::note info
インフォメーション
  クエリ (またはミューテーション) ファイルからクエリ自体以外のものをクライアントにインポートすることはできないため、共有の `validations.ts` ファイルを作成します。理由については [クエリの使用](https://blitzjs.com/docs/query-usage) および [ミューテーションの使用](https://blitzjs.com/docs/mutation-usage) を参照してください。
:::

次に `src/pages/questions/new.tsx` を開き、`CreateQuestion` を `src/questions/validations.ts` からインポートし、`QuestionForm` のスキーマとして設定します。また、`QuestionForm` の `initialValues` として `{{text: "", choices: []}}` を設定する必要があります。

```diff
// src/pages/questions/new.tsx

+ import {CreateQuestion} from

 "src/questions/validations"

<QuestionForm
  submitText="Create Question"
- // * フォームのバリデーションには zod スキーマを使用してください
- //  - ヒント: ミューテーションのスキーマを共有の `validations.ts` ファイルに抽出し、
- //         それをここでインポートして使用します
- // schema={createQuestion}
- // initialValues={{ }}
+ schema={CreateQuestion}
+ initialValues={{text: "", choices: []}}
  onSubmit={async (values) => {
    try {
      const question = await createQuestionMutation(values)
      router.push(Routes.ShowQuestionPage({ questionId: question.id }))
    } catch (error) {
      console.error(error)
      return {
        [FORM_ERROR]: error.toString(),
      }
    }
  }}
/>
```

#### 試してみてください

`localhost:3000/questions/new` にアクセスして、新しい質問と選択肢を作成できるか試してみてください！

## 選択肢のリスト表示 {#listing-choices}

ひと休みしましょう。ブラウザで `localhost:3000/questions` に戻り、作成したすべての質問を確認します。ここにも質問の選択肢をリスト表示してみませんか？まず、質問クエリをカスタマイズする必要があります。Prisma では、ネストされたリレーションをクエリすることをクライアントに手動で知らせる必要があります。`getQuestion.ts` と `getQuestions.ts` ファイルを次のように変更します：

```diff
// src/questions/queries/getQuestion.ts

const GetQuestion = z.object({
  // これは undefined 型を受け入れますが、実行時には必須です
  id: z.number().optional().refine(Boolean, "必須"),
})

export default resolver.pipe(
  resolver.zod(GetQuestion),
  resolver.authorize(),
  async ({id}) => {
-   const question = await db.question.findFirst({where: {id}})
+   const question = await db.question.findFirst({
+     where: {id},
+     include: {choices: true},
+   })

    if (!question) throw new NotFoundError()

    return question
  },
)
```

```diff
// src/questions/queries/getQuestions.ts

interface GetQuestionsInput
  extends Pick<
    Prisma.QuestionFindManyArgs,
    "where" | "orderBy" | "skip" | "take"
  > {}

export default resolver.pipe(
  resolver.authorize(),
  async ({where, orderBy, skip = 0, take = 100}: GetQuestionsInput) => {
    const {items: questions, hasMore, nextPage, count} = await paginate({
      skip,
      take,
      count: () => db.question.count({where}),
      query: (paginateArgs) =>
        db.question.findMany({
          ...paginateArgs,
          where,
          orderBy,
+         include: {choices: true},
        }),
    })

    return {
      questions,
      nextPage,
      hasMore,
      count,
    }
  },
)
```

次に、エディタでメインの質問ページ (`src/pages/questions/index.tsx`) に戻り、各質問の選択肢をリスト表示します。`QuestionsList` 内の `Link` の下に次のコードを追加します：

```diff
// src/pages/questions/index.tsx

// ...
{
  questions.map((question) => (
    <li key={question.id}>
      <Link href={Routes.ShowQuestionPage({ questionId: question.id })}>
        <a>{question.text}</a>
      </Link>
+     <ul>
+       {question.choices.map((choice) => (
+         <li key={choice.id}>
+           {choice.text} - {choice.votes} votes
+         </li>
+       ))}
+     </ul>
    </li>
  ))
}
// ...
```

アプリを再起動します — 開発サーバーを停止し、再び `yarn dev`、`npm dev`、または `pnpm dev` を実行します。ブラウザで `/questions` を確認してください。**魔法です！**

## これらの質問に投票できるようにしましょう！ {#let-people-vote-on-questions}

エディタで `src/pages/questions/[questionId].tsx` を開きます。まず、このページを少し改善します。

1. `<title>Question {question.id}</title>` を `<title>{question.text}</title>` に置き換えます。

2. `<h1>Question {question.id}</h1>` を `<h1>{question.text}</h1>` に置き換えます。

3. `pre` 要素を削除し、以前書いた選択肢のリストをコピーします：

```tsx
<ul>
  {question.choices.map((choice) => (
    <li key={choice.id}>
      {choice.text} - {choice.votes} votes
    </li>
  ))}
</ul>
```

ブラウザに戻ると、ページは次のようになっているはずです！

<img
  width="567"
  alt="スクリーンショット"
  src="https://user-images.githubusercontent.com/24858006/80387990-3c3d8b80-88a1-11ea-956a-5be85f1e8f12.png"
/>

#### さて、投票機能を追加しましょう！

まず `src/choices/mutations/updateChoice.ts` を開き、zod スキーマを更新し、投票のインクリメントを追加します。

```diff
const UpdateChoice = z
  .object({
    id: z.number(),
-   text: z.string(),
  })


export default resolver.pipe(
  resolver.zod(UpdateChoice),
  resolver.authorize(),
  async ({id, ...data}) => {
-   const choice = await db.choice.update({where: {id}, data})
+   const choice = await db.choice.update({
+     where: {id},
+     data: {votes: {increment: 1}},
+   })

    return choice
  },
)
```

次に `src/pages/questions/[questionId].tsx` に戻り、次の変更を行います：

`li` 内に `button` を追加します：

```tsx
<li key={choice.id}>
  {choice.text} - {choice.votes} votes
  <button>Vote</button>
</li>
```

次に、更新した `updateChoice` ミューテーションをインポートし、ページ内に `handleVote` 関数を作成します：

```diff
// src/pages/questions/[questionId].tsx
+import updateChoice from "src/choices/mutations/updateChoice"

//...

export const Question = () => {
  const router = useRouter()
  const questionId = useParam("questionId", "number")
  const [deleteQuestionMutation] = useMutation(deleteQuestion)
  const [question] = useQuery(getQuestion, {id: questionId})
+ const [updateChoiceMutation] = useMutation(updateChoice)
+
+ const handleVote = async (id: number) => {
+   try {
+     await updateChoiceMutation({id})
+     refetch()
+   } catch (error) {
+     alert("選択肢の更新中にエラーが発生しました " + JSON.stringify(error, null, 2))
+   }
+ }

  return (
```

次に、質問の `useQuery` 呼び出しを更新して、`handleVote` 内で使用する `refetch` 関数を返すようにします：

```diff
// src/pages/questions/[questionId].tsx

//...
- const [question] = useQuery(getQuestion, {id: questionId})
+ const [question, {refetch}] = useQuery(getQuestion, {id: questionId})
//...
```

最後に、新しい `button` にその関数を呼び出すように指示します！

```tsx
<button onClick={() => handleVote(choice.id)}>Vote</button>
```

最終的な `Question` コンポーネントは次のようになります：

```tsx
export const Question = () => {
  const router = useRouter()
  const questionId = useParam("questionId", "number")
  const [deleteQuestionMutation] = useMutation(deleteQuestion)
  const [question, { refetch }] = useQuery(getQuestion, {
    id: questionId,
  })
  const [updateChoiceMutation] = useMutation(updateChoice)

  const handleVote = async (id: number) => {
    try {
      await updateChoiceMutation({ id })
      refetch()
    } catch (error) {
      alert("選択肢の更新中にエラーが発生しました " + JSON.stringify(error, null, 2))
    }
  }

  return (
    <>
      <Head>
        <title>Question {question.id}</title>
      </Head>

      <div>
        <h1>{question.text}</h1>
        <ul>
          {question.choices.map((choice) => (
            <li key={choice.id}>
              {choice.text} - {choice.votes} votes
              <button onClick={() => handleVote(choice.id)}>Vote</button>
            </li>
          ))}
        </ul>

        <

Link href={Routes.EditQuestionPage({ questionId: question.id })}>
          <a>Edit</a>
        </Link>

        <button
          type="button"
          onClick={async () => {
            if (window.confirm("これを削除しますか？")) {
              await deleteQuestionMutation({ id: question.id })
              router.push(Routes.QuestionsPage())
            }
          }}
          style={{ marginLeft: "0.5rem" }}
        >
          Delete
        </button>
      </div>
    </>
  )
}
```

## 最後に、既存の質問の選択肢を編集できるようにしましょう {#edit-choices-for-question}

既存の質問の **Edit** ボタンをクリックすると、質問の作成と同じフォームが使用されていることがわかります。その部分は既に完了しています！ミューテーションを更新するだけです。

`src/questions/mutations/updateQuestion.ts` を開き、次の変更を行います：

```diff
// src/questions/mutations/updateQuestion.ts
import {resolver} from "blitz"
import db from "db"
import * as z from "zod"

const UpdateQuestion = z
  .object({
    id: z.number(),
    text: z.string(),
+   choices: z.array(
+     z.object({id: z.number().optional(), text: z.string()}),
+   ),
  })

export default resolver.pipe(
  resolver.zod(UpdateQuestion),
  resolver.authorize(),
  async ({id, ...data}) => {
-   const question = await db.question.update({where: {id}, data})
+   const question = await db.question.update({
+     where: {id},
+     data: {
+       ...data,
+       choices: {
+         upsert: data.choices.map((choice) => ({
+           // これは Prisma のバグのようです
+           // `|| 0` は必要ないはずです
+           where: {id: choice.id || 0},
+           create: {text: choice.text},
+           update: {text: choice.text},
+         })),
+       },
+     },
+     include: {
+       choices: true,
+     },
+   })

    return question
  },
)
```

[`upsert`](https://www.prisma.io/docs/reference/api-reference/prisma-client-reference#upsert) は特別な操作で、「このアイテムが存在する場合は更新し、そうでなければ作成する」ことを意味します。これはこのケースに最適です。質問の作成時に3つの選択肢を追加することをユーザーに強制しませんでした。そのため、後でユーザーが質問を編集して別の選択肢を追加した場合、それはここで作成されます。

## 結論 {#conclusion}

🥳 おめでとうございます！独自の Blitz アプリを作成しました！楽しんで遊んでみたり、友達と共有してみてください。このチュートリアルを終了したので、投票アプリをさらに改善してみませんか？例えば：

- スタイリングの追加
- 投票に関する統計情報の表示

Blitz コミュニティ全体とプロジェクトを共有したい場合は、Discord が最適です。

[discord.blitzjs.com](https://discord.blitzjs.com) を訪れてください。そして、**#built-with-blitz** チャンネルにリンクを投稿して、みんなと共有しましょう！
