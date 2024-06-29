---
title: useQuery
---

### 例 {#example}

BlitzアプリではデフォルトでReact Concurrent Modeが有効になっています。そのため、`<Suspense>`コンポーネントはロード状態の表示に、`<ErrorBoundary>`はエラーの表示に使用されます。必要に応じて、[React Concurrent Mode Docs](https://reactjs.org/docs/concurrent-mode-intro.html)を読むことができます。

```tsx
import { Suspense } from "react"
import { useQuery, useRouter, useParam } from "@blitzjs/rpc"
import getProject from "src/projects/queries/getProject"
import ErrorBoundary from "src/components/ErrorBoundary"

function Project() {
  const router = useRouter()
  const projectId = useParam("projectId", "number")
  const [project] = useQuery(getProject, { where: { id: projectId } })

  return <div>{project.name}</div>
}

function WrappedProject() {
  return (
    <div>
      <ErrorBoundary
        fallback={(error) => <div>Error: {JSON.stringify(error)}</div>}
      >
        <Suspense fallback={<div>Loading...</div>}>
          <Project />
        </Suspense>
      </ErrorBoundary>
    </div>
  )
}

export default WrappedProject
```

<br />

> さらなる例や使用例については、[クエリ使用法ドキュメント](./query-usage)を参照してください。

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
  }
] = useQuery(queryResolver, queryInputArguments, {
  cacheTime,
  enabled,
  initialData,
  initialDataUpdatedAt
  isDataEqual,
  keepPreviousData,
  notifyOnChangeProps,
  notifyOnChangePropsExclusions,
  onError,
  onSettled,
  onSuccess,
  queryKeyHashFn,
  refetchInterval,
  refetchIntervalInBackground,
  refetchOnMount,
  refetchOnReconnect,
  refetchOnWindowFocus,
  retry,
  retryOnMount,
  retryDelay,
  select,
  staleTime,
  structuralSharing,
  suspense,
  useErrorBoundary,
})
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

- `enabled: boolean`
  - これを`false`に設定すると、このクエリが自動的に実行されるのを無効にします。
  - [依存クエリ](./query-usage#dependent-queries)に使用できます。
- `retry: boolean | number | (failureCount: number, error: TError) => boolean`
  - `false`にすると、失敗したクエリはデフォルトでリトライしません。
  - `true`にすると、失敗したクエリは無限にリトライします。
  - `number`（例えば`3`）に設定すると、その数に達するまで失敗したクエリがリトライされます。
- `retryOnMount: boolean`
  - `false`に設定すると、エラーが含まれている場合でもマウント時にクエリはリトライされません。デフォルトは`true`です。
- `retryDelay: number | (retryAttempt: number, error: TError) => number`
  - この関数は`retryAttempt`整数と実際のエラーを受け取り、次の試行までの遅延をミリ秒単位で返します。
  - `attempt => Math.min(attempt > 1 ? 2 ** attempt * 1000 : 1000, 30 * 1000)`のような関数は指数バックオフを適用します。
  - `attempt => attempt * 1000`のような関数は線形バックオフを適用します。
- `staleTime: number | Infinity`
  - データが古くなったと見なされるまでのミリ秒単位の時間。これは、その定義されているフックのみに適用されます。
  - `Infinity`に設定すると、データは決して古くなりません。
- `cacheTime: number | Infinity`
  - 使用されていない/非アクティブなキャッシュデータがメモリに残る時間をミリ秒単位で設定します。クエリのキャッシュが未使用または非アクティブになると、この期間後にキャッシュデータはガベージコレクションされます。異なるキャッシュ時間が指定された場合、最長のものが使用されます。
  - `Infinity`に設定すると、ガベージコレクションが無効になります。
- `refetchInterval: false | number`
  - 任意
  - 数字に設定すると、すべてのクエリがこのミリ秒単位の周波数で継続的にリフェッチされます。
- `refetchIntervalInBackground: boolean`
  - 任意
  - `true`に設定すると、`refetchInterval`で継続的にリフェッチするように設定されたクエリは、そのタブ/ウィンドウがバックグラウンドにある間もリフェッチし続けます。
- `refetchOnMount: boolean | "always"`
  - 任意
  - デフォルトは`true`
  - `true`に設定すると、データが古い場合にクエリはマウント時にリフェッチされます。
  - `false`に設定すると、クエリはマウント時にリフェッチされません。
  - `"always"`に設定すると、クエリは常にマウント時にリフェッチされます。
- `refetchOnWindowFocus: boolean | "always"`
  - 任意
  - デフォルトは`true`
  - `true`に設定すると、データが古い場合にウィンドウがフォーカスされたときにクエリがリフェッチされます。
  - `false`に設定すると、ウィンドウがフォーカスされたときにクエリはリフェッチされません。
  - `"always"`に設定すると、ウィンドウがフォーカスされたときに常にクエリがリフェッチされます。
- `refetchOnReconnect: boolean | "always"`
  - 任意
  - デフォルトは`true`
  - `true`に設定すると、データが古い場合に再接続時にクエリがリフェッチされます。
  - `false`に設定すると、再接続時にクエリはリフェッチされません。
  - `"always"`に設定すると、再接続時に常にクエリがリフェッチされます。
- `notifyOnChangeProps: string[] | "tracked"`
  - 任意
  - 設定すると、リストされたプロパティが変更されたときにのみコンポーネントが再レンダリングされます。
  - 例えば`['data', 'error']`に設定すると、`data`または`error`プロパティが変更されたときにのみコンポーネントが再レンダリングされます。
  - `"tracked"`に設定すると、プロパティへのアクセスが追跡され、追跡されていたプロパティが変更されたときにのみコンポーネントが再レンダリングされます。
- `notifyOnChangePropsExclusions: string[]`
  - 任意
  - 設定すると、リストされたプロパティが変更されてもコンポーネントは再レンダリングされません。
  - 例えば`['isStale']`に設定すると、`isStale`プロパティが変更されてもコンポーネントは再レンダリングされません。
- `onSuccess: (data: TData) => void`
  - 任意
  - クエリが新しいデータを正常に取得するたびにこの関数が実行されます。
- `onError: (error: TError) => void`
  - 任意
  - クエリがエラーに遭遇した場合にこの関数が実行され、エラーが渡されます。
- `onSettled: (data?: TData, error?: TError) => void`
  - 任意
  - クエリが正常に取得されたかエラーが発生した場合にこの関数が実行され、データまたはエラーが渡されます。
- `select: (data: TData) => unknown`
  - 任意
  - クエリ関数によって返されたデータの一部を変換または選択するために使用できます。
- `suspense: boolean`
  - 任意
  - これを`false`に設定すると、サスペンスモードが無効になります。
  - `true`に設定すると、`status === 'loading'`のときに`useQuery`はサスペンドします。
  - `true`に設定すると、`status === 'error'`のときに実行時エラーがスローされます。
- `initialData: TData | () => TData`
  - 任意
  - 設定すると、この値がクエリキャッシュの初期データとして使用されます（クエリがまだ作成またはキャッシュされていない限り）。
  - 関数に設定すると、その関数は共有/ルートクエリ初期化の間に**一度だけ**呼び出され、初期データを同期的に返すことが期待されます。
  - 初期データはデフォルトで古くなっていますが、`staleTime`が設定されている場合を除きます。
  - `initialData`は**キャッシュに保存されます**。
- `initialDataUpdatedAt: number | (() => number | undefined)`
  - 任意
  - 設定すると、この値が`initialData`自体が最後に更新された時点（ミリ秒）として使用されます。
- `placeholderData: TData | () => TData`
  - 任意
  - 設定すると、この値がこの特定のクエリオブザーバがまだ`loading`状態であり、初期データが提供されていないときのプレースホルダデータとして使用されます。
  - `placeholderData`は**キャッシュされません**。
- `keepPreviousData: boolean`
  - 任意
  - デフォルトは`false`。
  - 設定すると、クエリキーが変更されたために新しいデータを取得しているときに以前の`data`を保持します。
- `structuralSharing: boolean`
  - 任意
  - デフォルトは`true`。
- `useErrorBoundary: boolean`
  - 任意
  - デフォルトは全局クエリ設定の`useErrorBoundary`値で、デフォルトは`false`。
  - クエリエラーがレンダリングフェーズでスローされ、最も近いエラーバウンダリに伝播することを望む場合は、これを`true`に設定します。

### 戻り値 {#returns}

`[queryResult, queryExtras]`

##### `queryResult: TData`

- デフォルトは`undefined`。
- クエリが最後に正常に解決したデータ。

##### `queryExtras: Object`

- `status: String`
  - 次のように設定されます:
    - `idle`: クエリがアイドル状態。これは`enabled: false`で初期化され、初期データが利用できない場合にのみ発生します。
    - `loading`: クエリが「ハード」なロード状態。この状態ではキャッシュデータがなく、クエリが現在取得中であることを意味し、例えば`isFetching === true`。
    - `error`: クエリ試行がエラーになった場合。対応する`error`プロパティには試行された取得から受け取ったエラーが含まれます。
    - `success`: クエリがエラーなく応答を受信し、そのデータが表示可能であることを示します。対応する`data`プロパティには正常に取得されたデータが含まれます。または、クエリの`enabled`プロパティが`false`に設定され、まだ取得されていない場合、`data`は初期化時にクエリに最初に供給された`initialData`です。
- `isIdle: boolean`
  - 上記の`status`変数から派生したboolean値で、利便性のために提供されます。
- `isLoading: boolean`
  - 上記の`status`変数から派生したboolean値で、利便性のために提供されます。
- `isSuccess: boolean`
  - 上記の`status`変数から派生したboolean値で、利便性のために提供されます。
- `isError: boolean`
  - 上記の`status`変数から派生したboolean値で、利便性のために提供されます。
- `isLoadingError: boolean`
  - クエリが初めて取得中に失敗した場合は`true`。
- `isRefetchError: boolean`
  - クエリがリフェッチ中に失敗した場合は`true`。
- `data: TData`
  - デフォルトは`undefined`。
  - クエリが最後に正常に解決したデータ。
- `dataUpdatedAt: number`
  - クエリが最後に`status`を`"success"`として返した時刻のタイムスタンプ。
- `error: null | TError`
  - デフォルトは`null`。
  - クエリに対するエラーオブジェクト（エラーが発生した場合）。
- `errorUpdatedAt: number`
  - クエリが最後に`status`を`"error"`として返した時刻のタイムスタンプ。
- `isStale: boolean`
  - キャッシュ内のデータが無効化されている場合、または与えられた`staleTime`より古い場合は`true`。
- `isPlaceholderData: boolean`
  - 表示されているデータがプレースホルダデータである場合は`true`。
- `isPreviousData: boolean`
  - `keepPreviousData`が設定されており、以前のクエリのデータが返された場合は`true`。
- `isFetched: boolean`
  - クエリが取得された場合は`true`。
- `isFetchedAfterMount: boolean`
  - コンポーネントがマウントされた後にクエリが取得された場合は`true`。
  - このプロパティは、以前にキャッシュされたデータを表示しないために使用できます。
- `isFetching: boolean`
  - `enabled`が`false`に設定されている限り、デフォルトは`true`。
  - クエリが現在取得中である場合は`true`。バックグラウンド取得も含まれます。
- `failureCount: number`
  - クエリの失敗回数。
  - クエリが失敗するたびに増加します。
  - クエリが成功すると`0`にリセットされます。
- `refetch: (options: { throwOnError: boolean, cancelRefetch: boolean }) => Promise<UseQueryResult>`
  - クエリを手動でリフェッチするための関数。
  - クエリがエラーを起こした場合、そのエラーはログに記録されるだけです。エラーをスローしたい場合は、`throwOnError: true`オプションを渡します。
  - `cancelRefetch`が`true`の場合、現在のリクエストは新しいリクエストが行われる前にキャンセルされます。
- `remove: () => void`
  - キャッシュからクエリを削除するための関数。
- `setQueryData()` - `Function(newData, opts) => Promise<void>`
  - クエリのキャッシュを手動で更新するための関数。
  - `newData`は新しいデータのオブジェクトか、古いデータを受け取り新しいデータを返す関数である必要があります。
  - フォームを送信した後、キャッシュを即座に更新するためによく使用されます。
  - キャッシュを更新した後、データが正しいことを確認するために自動的に`refetch()`が呼び出されます。2番目の引数としてオプションオブジェクト`{refetch: false}`を渡すことにより、リフェッチを無効にすることができます。
  - `setQueryData()`の使用例については[Blitz mutation usage docs](./mutation-usage#setQueryData)を参照してください。
