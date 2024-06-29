---
title: クエリの使用
sidebar_label: クエリを使用
---

## React コンポーネント内で {#in-a-react-component}

[Blitz のクエリ](./query-resolvers)の一つを使用するには、`useQuery` フックを次のように呼び出します:

1. クエリリゾルバ
2. クエリ関数の入力引数
3. 必要に応じて、設定オブジェクト。

それはこのタプル配列、`[queryResult, queryExtras]`を返します。この中で `queryResult` はクエリ関数から返されたものと全く同じです。

`useQuery` は [`react-query`](https://github.com/tannerlinsley/react-query) 上に構築されているため、自動キャッシュや再検証などの素晴らしい機能が自動的に提供されます。

```ts
import { useQuery } from "@blitzjs/rpc"
import getProject from "src/projects/queries/getProject"

function App() {
  const [project] = useQuery(getProject, { where: { id: 1 } })
}
```

詳細については、[`useQuery` のドキュメント](./use-query)を参照してください。

:::message
サーバーコードをコンポーネントにインポートしているのに、どうやって動作するのかと思うかもしれません: ビルド時に、直接関数のインポートがネットワーク呼び出しに置き換えられます。したがって、クエリ関数のコードはクライアントコードには含まれません。
:::

### 覚えておくべきデフォルト設定 {#defaults-to-keep-in-mind}

初めから、`useQuery` およびその他のクエリフックは**積極的だが合理的な**デフォルト設定が構成されています。**これらのデフォルトは、時には新しいユーザーにとって不意をつくことがあるかもしれませんし、学習やデバッグを困難にすることがあります。**:

- _現在画面に表示されている_クエリの結果は、解決後すぐに「古い」ものとなり、再度表示または使用されるときに自動的にバックグラウンドで再取得されます。これを変更するには、クエリのデフォルトの`staleTime`を`0`ミリ秒以外のものに変更できます。
- 使用されなくなったクエリ結果（クエリのすべてのインスタンスがアンマウントされる）は、再度使用される場合に備えてデフォルトでは5分間キャッシュされ、その後ガベージコレクションされます。これを変更するには、クエリのデフォルトの`cacheTime`を`1000 * 60 * 5`ミリ秒以外のものに変更できます。
- 古いクエリは、**ブラウザウィンドウがユーザーによって再フォーカスされたり、ブラウザが再接続されたりしたときに**自動的にバックグラウンドで再取得されます。これを無効にするには、クエリ内の`refetchOnWindowFocus`および`refetchOnReconnect`オプションを使用します。
- ネットワークエラーのために本番環境で失敗したクエリは、**指数的バックオフ遅延を伴う3回の再試行**が自動的に行われた後、UIにエラーを表示します。これを変更するには、クエリのデフォルトの`retry`および`retryDelay`オプションを変更します。
- クエリ結果はデフォルトで深い比較が行われ、データが実際に変更されたかどうかが検出されます。変更がなければ、データ参照は変更されず、useMemoやuseCallbackとの関連で値の安定化が向上します。ここで使用されるデフォルトの深い比較関数（`config.isDataEqual`）は、JSON互換のプリミティブのみを比較することをサポートします。クエリ応答で非JSON互換の値を扱っている場合や、深い比較関数でパフォーマンスの問題が発生している場合、この比較を無効にする（`config.isDataEqual = () => false`）か、よりニーズに合わせてカスタマイズするべきです。

### オプション {#options}

利用可能なオプションの完全なリストについては、[`useQuery` のドキュメント](./use-query)を参照してください。

### 従属クエリ {#dependent-queries}

従属クエリは、前のクエリが終了するまで実行されないクエリです。これを行うには、クエリが動作する準備ができたときに知らせるために enabled オプションを使用します:

```tsx
const [user] = useQuery(getUser, { where: { id: props.query.id } })
const [projects] = useQuery(getProjects, { where: { userId: user.id } }, { enabled: user }))
```

### ページネーション {#pagination}

[`usePaginatedQuery`](./use-paginated-query) フックを使用します。

### 無限スクロール {#infinite-loading}

[`useInfiniteQuery`](./use-infinite-query) フックを使用します。

### プリフェッチ {#prefetching}

すべてのクエリは自動的にキャッシュされるため、次のどちらも `getProject` クエリをキャッシュします。

```ts
import { invoke } from "@blitzjs/rpc"

const project = await invoke(getProject, { where: { id: props.id } })
```

```tsx
<a onMouseEnter={() => invoke(getProject, { where: { id: props.id } })}>
  View Project
</a>
```

Blitz はサーバー上で複数のクエリをプリフェッチし、そのクエリを内部のクエリクライアントにデハイドレートすることもサポートしています。これにより、Blitz はページ読み込み時に即座に利用できるマークアップを事前にレンダリングできます。ブラウザにJavaScriptが利用可能になると、Blitzはこれらのクエリをハイドレートし、サーバーでレンダリングされた後に古くなった場合は再取得します。

次の例は、Blitz におけるプリフェッチの動作を示しています。

```tsx
import { gSSP } from "src/blitz-server"
import getProject from "src/projects/queries/getProject"

export const getServerSideProps = gSSP(async ({ ctx }) => {
  // 重要: prefetchBlitzQuery の第二引数は、下記の useQuery の第二引数と正確に一致する必要があります
  await context.prefetchBlitzQuery(getProject, {
    where: { id: context.params?.projectId },
  })

  return { props: {} }
})

function ProjectPage(props) {
  const [project] = useQuery(getProject, { where: { id: props.id } })
  return <div>{project.name}</div>
}

export default ProjectPage
```

データがユーザーやリクエスト間で共有されないようにするために、新しいクエリクライアントが各ページリクエストのために作成されます。`ctx` オブジェクトで利用可能な `prefetchBlitzQuery` メソッドを呼び出し、リゾルバと関連する入力引数を渡すことで、データをプリフェッチできます。データが取得されると、Blitz はクエリをクエリクライアントにデハイドレートする処理を行います。

### クエリのキャンセル {#query-cancellation}

Blitz は React Query のデフォルトのクエリキャンセル方法をサポートしています。詳細はこちらで読むことができます:
[here](https://react-query-v2.tanstack.com/guides/query-cancellation).

また、`cancelQueries` メソッドを使用して手動でクエリをキャンセルすることもできます:

```tsx
import { getQueryClient, getQueryKey, useQuery } from "@blitzjs/rpc"

const [data] = useQuery(getTodos)

return (
  <button
    onClick={(e) => {
      e.preventDefault()
      getQueryClient.cancelQueries(getQueryKey(getTodos))
    }}
  >
    キャンセル
  </button>
)
```

## サーバー上で {#on-the-server}

:::message
`getStaticProps` または `getServerSideProps` では、クエリおよびミューテーション関数を直接呼び出すことができます。
:::

### `getStaticProps` {#get-static-props}

```tsx
import getProject from "src/projects/queries/getProject"
import { gSP } from "src/blitz-server"

export const getStaticProps = gSP(async ({ params, ctx }) => {
  const project = await getProject({
    where: { id: context.params?.projectId },
  })
  return { props: { project } }
})

function ProjectPage({ project }) {
  return <div>{project.name}</div>
}

export default ProjectPage
```

### `getServerSideProps` {#get-server-side-props}

```tsx
import getProject from "src/projects/queries/getProject"
import { gSSP } from "src/blitz-server"

export const getServerSideProps = gSSP(async ({ params, ctx }) => {
  const project = await getProject({
    where: { id: params.projectId },
  })

  return { props: { project } }
})

function ProjectPage({ project }) {
  return <div>{project.name}</div>
}

export default ProjectPage
```
