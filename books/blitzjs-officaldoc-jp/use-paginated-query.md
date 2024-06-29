    ===
    ---
title: usePaginatedQuery
sidebar_label: usePaginatedQuery
---

`useQuery`の代わりに`usePaginatedQuery`を使用して、大量のデータを小さな連続する区間に分割して表示する場合に便利です。例えば、多くの番号付きページがあり、最初に最初の3ページを表示したいとします。「次のページを表示」ボタンをクリックすると、ユーザーは次の3ページだけを見ることができるようにします。この場合、以下の例のように`usePaginatedQuery`を使用できます。

### 簡単な前/次の例 {#prev-next-example}

```tsx
import { usePaginatedQuery } from "@blitzjs/rpc"
import { Routes } from "@blitzjs/next"
import { useRouter } from "next/router"
import Link from "next/link"
import getProjects from "src/products/queries/getProjects"

const ITEMS_PER_PAGE = 3

const Projects = () => {
  const router = useRouter()
  const page = Number(router.query.page) || 0
  const [{ projects, hasMore }] = usePaginatedQuery(getProjects, {
    orderBy: { id: "asc" },
    skip: ITEMS_PER_PAGE * page,
    take: ITEMS_PER_PAGE,
  })

  const goToPreviousPage = () =>
    router.push({ query: { page: page - 1 } })
  const goToNextPage = () => router.push({ query: { page: page + 1 } })

  return (
    <div>
      {projects.map((project) => (
        <p key={project.id}>
          <Link href={Routes.Project({ handle: project.handle })}>
            <a>{project.name}</a>
          </Link>
        </p>
      ))}
      <button disabled={page === 0} onClick={goToPreviousPage}>
        Previous
      </button>
      <button disabled={!hasMore} onClick={goToNextPage}>
        Next
      </button>
    </div>
  )
}
```

これに対応するクエリは以下の通りです：

```ts
import { paginate } from "blitz"
import { resolver } from "@blitzjs/rpc"
import db, { Prisma } from "db"

interface GetProjectsInput
  extends Pick<
    Prisma.ProjectFindManyArgs,
    "where" | "orderBy" | "skip" | "take"
  > {}

export default resolver.pipe(
  resolver.authorize(),
  async ({ where, orderBy, skip = 0, take = 100 }: GetProjectsInput) => {
    const {
      items: projects,
      hasMore,
      nextPage,
      count,
    } = await paginate({
      skip,
      take,
      count: () => db.project.count({ where }),
      query: (paginateArgs) =>
        db.project.findMany({ ...paginateArgs, where, orderBy }),
    })

    return {
      projects,
      nextPage,
      hasMore,
      count,
    }
  }
)
```

`usePaginatedQuery`は複数のページ間をナビゲートする機能もサポートしています。例えば、多くの番号付きページがあり、ページ範囲から特定のページを選択するナビゲーションコントロールを表示したい場合です。この場面では、以下の例のように`usePaginatedQuery`を使用できます。

### 範囲ページネーションの例 {#range-example}

```tsx
import { usePaginatedQuery } from "@blitzjs/rpc"
import { Routes } from "@blitzjs/next"
import { useRouter } from "next/router"
import Link from "next/link"
import getProjects from "src/products/queries/getProjects"

const Projects = () => {
  const router = useRouter()
  const page = Number(router.query.page) || 0
  const [{ projects, count, pageCount, from, to }] = usePaginatedQuery(getProjects, {
    orderBy: { id: "asc" },
    skip: ITEMS_PER_PAGE * page,
    take: ITEMS_PER_PAGE,
  })
  
  function range(start: number, end: number) {
    const length = end - start + 1;
    return Array.from({ length }, (_, index) => index + start);
  }

  return (
    <div>
      {projects !== undefined && 
      <>
        {projects.map((p) =>
          <div key={p.id}>
            <p>name: {p.name}</p>            
            <hr />
          </div>
        )}
        <div style={{display:"flex", flexDirection:"column"}}>                
          <nav style={{display:"flex"}}>
              <button
                onClick={() => router.push({ query: { page: page - 1 } })}
                disabled={page == 0}              
              >
                <span>&lt; Previous</span>
              </button>
              {range(1, pageCount).map((page: number, idx: number) => {
                return (
                  <button
                    key={idx}
                    style={{"backgroundColor": idx === page ? "#0047AB": "#808080", color:"#FFF"}}
                    onClick={() => {
                      router.push({ query: { page: page - 1 } })
                    }}                  
                  >
                    {page}
                  </button>
                )
              })}
              <button
                onClick={() => router.push({ query: { page: page + 1 } })}
                disabled={page == (pageCount-1)}              
              >
                <span>Next &gt;</span>
              </button>
          </nav>
          <p>
            <span>Showing </span>
            <span>{from}</span>
            <span> to </span>
            <span>{to}</span>
            <span> of </span>
            <span>{count}</span>
            <span> results</span>
          </p>
        </div>    
      </>}            
    </div>
  )
}
```

これに対応するクエリは以下の通りです：

```ts
import { paginate } from "blitz"
import { resolver } from "@blitzjs/rpc"
import db, { Prisma } from "db"

interface GetProjectsInput
  extends Pick<
    Prisma.ProjectFindManyArgs,
    "where" | "orderBy" | "skip" | "take"
  > {}

export default resolver.pipe(
  resolver.authorize(),
  async ({ where, orderBy, skip = 0, take = 100 }: GetProjectsInput) => {
    const {
      items: projects,
      pageCount,
      from,
      to,
      count,
    } = await paginate({
      skip,
      take,
      count: () => db.project.count({ where }),
      query: (paginateArgs) =>
        db.project.findMany({ ...paginateArgs, where, orderBy }),
    })

    return {
      projects,
      pageCount,
      from,
      to,
      count,
    }
  }
)
```

## API {#api}

```js
const [
  queryResult,
  {
    dataUpdatedAt,
    error,
    errorUpdatedAt,
    failureCount,
    isError,
    isFetched,
    isFetchedAfterMount,
    isFetching,
    isIdle,
    isLoading,
    isLoadingError,
    isPlaceholderData,
    isPreviousData,
    isRefetchError,
    isStale,
    isSuccess,
    refetch,
    remove,
    status,
    setQueryData,
  },
] = usePaginatedQuery(queryResolver, queryInputArguments, options)
```

### 引数 {#arguments}

- `queryResolver:` [Blitz query resolver](./query-resolvers)
  - **必須**
- `queryInputArguments`
  - **必須**
  - `queryResolver`に渡される引数
- `options`
  - 任意

### オプション {#options}

オプションは[useQueryフック](./use-query)のオプションと同一です。

### 戻り値 {#returns}

`[queryResult, queryExtras]`

##### `queryResult: TData`

- デフォルトは`undefined`。
- クエリの最後に正常に解決されたデータ。

##### `queryExtras: Object`

- `status: String`
  - 以下のいずれかの状態になる：
    - `idle`：クエリがアイドル状態。これは、クエリが`enabled: false`で初期化され、初期データがない場合に発生
    - `loading`：クエリが「ハード」ローディング状態。これは、キャッシュデータがなく、クエリが現在フェッチ中であることを意味し、例えば`isFetching === true`
    - `error`：クエリの試行がエラーを返した場合。この場合の対応する`error`プロパティには、試行されたフェッチから受け取ったエラーが含まれる
    - `success`：クエリがエラーなしで応答を受信し、データの表示準備が整っている場合。この場合の対応する`data`プロパティには成功したフェッチから受け取ったデータが含まれる。クエリの`enabled`プロパティが`false`に設定されており、まだフェッチされていない場合、`data`には初回の`initialData`が含まれる。
- `isIdle: boolean`
  - 上記の`status`変数から派生したブール値であり、利便性のために提供される。
- `isLoading: boolean`
  - 上記の`status`変数から派生したブール値であり、利便性のために提供される。
- `isSuccess: boolean`
  - 上記の`status`変数から派生したブール値であり、利便性のために提供される。
- `isError: boolean`
  - 上記の`status`変数から派生したブール値であり、利便性のために提供される。
- `isLoadingError: boolean`
  - クエリが初めてフェッチ中に失敗した場合に`true`になる。
- `isRefetchError: boolean`
  - クエリがリフェッチ中に失敗した場合に`true`になる。
- `data: TData`
  - デフォルトは`undefined`。
  - クエリの最後に正常に解決されたデータ。
- `dataUpdatedAt: number`
  - クエリが最後に`status`を`"success"`として返した時のタイムスタンプ。
- `error: null | TError`
  - デフォルトは`null`
  - クエリのエラーオブジェクトで、エラーが投げられた場合に含まれる。
- `errorUpdatedAt: number`
  - クエリが最後に`status`を`"error"`として返した時のタイムスタンプ。
- `isStale: boolean`
  - キャッシュ内のデータが無効化された場合、またはデータが指定された`staleTime`より古い場合に`true`になる。
- `isPlaceholderData: boolean`
  - 表示されているデータがプレースホルダーデータである場合に`true`になる。
- `isPreviousData: boolean`
  - 前回のクエリからのデータが返された場合に`true`になる。
- `isFetched: boolean`
  - クエリがフェッチされた場合に`true`になる。
- `isFetchedAfterMount: boolean`
  - クエリがコンポーネントのマウント後にフェッチされた場合に`true`になる。
  - このプロパティは、以前にキャッシュされたデータを表示しないために使用できる。
- `isFetching: boolean`
  - デフォルトは`true`。ただし`enabled`が`false`に設定されている限り。
  - クエリが現在フェッチ中（バックグラウンドフェッチを含む）の場合に`true`になる。
- `failureCount: number`
  - クエリの失敗回数。
  - クエリが失敗するたびにインクリメントされる。
  - クエリが成功するたびに0にリセットされる。
- `refetch: (options: { throwOnError: boolean, cancelRefetch: boolean }) => Promise<UseQueryResult>`
  - クエリを手動でリフェッチするための関数。
  - クエリがエラーを返した場合、エラーはログに記録されるだけである。エラーをスローさせたい場合は、`throwOnError: true`オプションを渡す。
  - `cancelRefetch`が`true`の場合、現在のリクエストは新しいリクエストが行われる前にキャンセルされる。
- `remove: () => void`
  - キャッシュからクエリを削除するための関数。
- `setQueryData()` - `Function(newData, opts) => Promise<void>`
  - クエリのキャッシュを手動で更新するための関数。
  - `newData`は新しいデータのオブジェクトまたは古いデータを受け取り新しいデータを返す関数として渡せる。
  - これはフォーム送信後にキャッシュを即座に更新するためによく使用される。
  - キャッシュの更新後、データの正確性を保証するために自動的に`refetch()`を呼び出す。オプションオブジェクトとして2番目の引数に`{refetch: false}`を渡すことでリフェッチを無効にする。
  - `setQueryData()`の使用例については[Blitzミューテーション使用ドキュメント](./mutation-usage#setQueryData)を参照。

    ===