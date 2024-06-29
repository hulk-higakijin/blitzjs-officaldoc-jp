    ```
    ---
    title: useInfiniteQuery
    sidebar_label: useInfiniteQuery
    ---

    `useQuery` の代わりに `useInfiniteQuery` を使用して、大量のデータを徐々に表示することができます。例えば、多くのページがあり、最初に10ページを表示したい場合、`Show more pages` をクリックすると、ユーザーは前よりも3ページ多く表示されます。この場合、次の例のように `useInfiniteQuery` を使用できます。

    ### 例 {#example}

    ```tsx
    import { useInfiniteQuery } from "@blitzjs/rpc"
    import getProjects from "src/projects/queries/getProjects"

    function Projects(props) {
      const [
        projectPages,
        { isFetching, isFetchingNextPage, fetchNextPage, hasNextPage },
      ] = useInfiniteQuery(
        getProjects,
        (page = { take: 3, skip: 0 }) => page,
        {
          getNextPageParam: (lastPage) => lastPage.nextPage,
        }
      )
      return (
        <>
          {projectPages.map((page, i) => (
            <React.Fragment key={i}>
              {page.projects.map((project) => (
                <p key={project.id}>{project.name}</p>
              ))}
            </React.Fragment>
          ))}
          <div>
            <button
              onClick={() => fetchNextPage()}
              disabled={!hasNextPage || !!isFetchingNextPage}
            >
              {isFetchingNextPage
                ? "Loading more..."
                : hasNextPage
                ? "Load More"
                : "Nothing more to load"}
            </button>
          </div>
          <div>
            {isFetching && !isFetchingNextPage ? "Fetching..." : null}
          </div>
        </>
      )
    }
    ```

    以下はそれに対応するクエリです：

    ```ts
    import { resolver } from "@blitzjs/rpc"
    import { paginate } from "blitz"
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

    ## API {#api}

    ```js
    const [
      pagesOfQueryResults,
      {
        pageParams,
        fetchNextPage,
        fetchPreviousPage,
        hasNextPage,
        hasPreviousPage,
        isFetchingNextPage,
        isFetchingPreviousPage,
        ...extras
      },
    ] = useInfiniteQuery(queryResolver, getQueryInputArguments, {
      ...options,
      getNextPageParam: (lastPage, allPages) => lastPage.nextPage,
      getPreviousPageParam: (firstPage, allPages) => firstPage.nextPage,
    })
    ```

    ### 引数 {#arguments}

    - `queryResolver:` [Blitz クエリリゾルバ](./query-resolvers)
      - **必須**
    - `getQueryInputArguments: (fetchNextPageVariable) => queryInputArguments`
      - **必須**
      - 現在のページオプションを受け取り、`queryInputArguments` を返す関数
      - 最初のページロード時、`fetchNextPageVariable` は `undefined`
      - 以降のページでは、`fetchNextPageVariable` は `getNextPageParam()` から返されるもの
    - `options`
      - オプション

    ### オプション {#options}

    オプションは [useQuery フック](./use-query#options) と同じですが、以下のものが追加されています：

    - `getNextPageParam: (lastPage, allPages) => unknown | undefined`
      - 新しいデータが取得されると、この関数は無限リストの最後のページと全ページの配列を受け取ります。
      - それは `getQueryInputArguments()` に引数として渡される単一の変数を返す必要があります。
      - 次のページが使用可能でないことを示すには `undefined` を返します。
    - `getPreviousPageParam: (firstPage, allPages) => unknown | undefined`
      - 新しいデータが取得されると、この関数は無限リストの最初のページと全ページの配列を受け取ります。
      - それは `getQueryInputArguments()` に引数として渡される単一の変数を返す必要があります。
      - 前のページが使用可能でないことを示すには `undefined` を返します。

    ### 戻り値 {#returns}

    `\[pagesOfQueryResults, queryExtras\]`

    ##### `pagesOfQueryResults: TData[]`

    - 全てのページを含む配列。

    ##### `queryExtras: Object`

    `useInfiniteQuery` のクエリエクストラは [useQuery フック](./use-query#returns) と同じですが、以下のものが追加されています：

    - `pageParams: unknown[]`
      - 全てのページパラメータを含む配列。
    - `isFetchingNextPage: boolean`
      - `fetchNextPage` で次のページをフェッチ中は `true` になる。
    - `isFetchingPreviousPage: boolean`
      - `fetchPreviousPage` で前のページをフェッチ中は `true` になる。
    - `fetchNextPage: (options?: FetchNextPageOptions) => Promise<UseInfiniteQueryResult>`
      - 次の "ページ" の結果をフェッチするこの関数。
      - `options.pageParam: unknown` は、`getNextPageParam` を使用せずに手動でページパラメータを指定できます。
    - `fetchPreviousPage: (options?: FetchPreviousPageOptions) => Promise<UseInfiniteQueryResult>`
      - 前の "ページ" の結果をフェッチするこの関数。
      - `options.pageParam: unknown` は、`getPreviousPageParam` を使用せずに手動でページパラメータを指定できます。
    - `hasNextPage: boolean`
      - 次のページをフェッチする必要がある場合（`getNextPageParam` オプションを介して知られる）、`true` になります。
    - `hasPreviousPage: boolean`
      - 前のページをフェッチする必要がある場合（`getPreviousPageParam` オプションを介して知られる）、`true` になります。
    - `setQueryData()` - `Function(newData, opts) => void`
      - クエリのキャッシュを手動で更新するための関数。
      - `newData` は新しいデータのオブジェクトまたは古いデータを受け取り新しいデータを返す関数。
      - フォーム送信後にキャッシュをすぐに更新するためによく使用されます。
      - キャッシュを更新した後、データが正しいことを確認するために自動的に `refetch()` を呼び出します。第二引数として `{refetch: false}` のオプションオブジェクトを渡してリフェッチを無効にします。
      - `setQueryData()` の例については [Blitz ミューテーション使用方法のドキュメント](./mutation-usage#setQueryData) を参照してください。

    ```