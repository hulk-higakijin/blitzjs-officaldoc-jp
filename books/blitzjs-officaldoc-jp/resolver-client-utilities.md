    ---
title: クライアントユーティリティ
sidebar_label: クライアントユーティリティ
---

## `invoke` {#invoke}

これはコンポーネントからクエリやミューテーションを命令的に呼び出すためのものです。

### 例 {#example}

```tsx
import { invoke } from "@blitzjs/rpc"
import getProducts from "src/products/queries/getProducts"

const Page = function ({ products }) {
  return (
    <div>
      <button
        onClick={async () => {
          const products = await invoke(getProducts, {
            where: { orgId: 1 },
          })
        }}
      >
        Get Products
      </button>
    </div>
  )
}
export default Page
```

## API {#api}

```js
const results = await invoke(resolver, resolverInputArguments)
```

### 引数 {#arguments}

- `resolver:` Blitzの[クエリリゾルバ](./query-resolvers) または [ミューテーションリゾルバ](./mutation-resolvers)
  - **必須**
- `resolverInputArguments`
  - **必須**
  - `resolver`に渡される引数

### 戻り値 {#returns}

- `results`
  - リゾルバから返される正確な結果

## `invalidateQuery` {#invalidate-query}

これは特定のクエリを無効にするためのものです。クエリのデータが古いことが確実で、自動再フェッチを待ちたくない場合に`invalidateQuery`を使用します。

### 例 {#example-1}

```tsx
import { invalidateQuery } from "@blitzjs/rpc" // highlight-line
import getProducts from "src/products/queries/getProducts"

const Page = function ({ products }) {
  return (
    <div>
      <button
        onClick={() => {
          invalidateQuery(getProducts) // highlight-line
        }}
      >
        Invalidate
      </button>
    </div>
  )
}
export default Page
```

## API {#api-1}

```js
const refetchedQueries = await invalidateQuery(resolver, inputArgs?)
```

### 引数 {#arguments-1}

- `resolver:` Blitzの[クエリリゾルバ](./query-resolvers) または [ミューテーションリゾルバ](./mutation-resolvers)
  - **必須**
- `inputArgs`
  - 任意
  - `resolver`に渡される引数
  - これを提供しない場合、すべての`resolver`クエリが無効になります
  - これを提供すると、`resolver`の該当する`inputArgs`クエリのみが無効になります

### 戻り値 {#returns-1}

この関数は、すべてのクエリが再フェッチされ終わった時に解決されるプロミスを返します。

## `getQueryKey` {#get-query-key}

これはクエリのクエリキーを取得するためのものです。`queryClient`はクエリキーに基づいてクエリを管理します。詳細な説明は[`React Query`](https://react-query.tanstack.com/guides/query-keys)のドキュメントを参照してください。

### 例 {#example-2}

```tsx
import { getQueryKey, getQueryClient } from "@blitzjs/rpc"
import getProducts from "src/products/queries/getProducts"

const Page = function ({ products }) {
  return (
    <div>
      <button
        onClick={async () => {
          const queryKey = getQueryKey(getProducts)
          getQueryClient().invalidateQueries(queryKey)

          const queryKey = getQueryKey(getProducts, {
            where: { ordId: 1 },
          })
          getQueryClient().invalidateQueries(queryKey)
        }}
      >
        Invalidate
      </button>
    </div>
  )
}
export default Page
```

## `getInfiniteQueryKey` {#get-infinite-query-key}

これは**無限**クエリのクエリキーを取得するためのものです。`queryClient`はクエリキーに基づいてクエリを管理します。詳細な説明は[`React Query`](https://react-query.tanstack.com/guides/query-keys)のドキュメントを参照してください。

### 例 {#example-3}

```tsx
import { getQueryKey, getQueryClient } from "@blitzjs/rpc"
import getInfiniteProducts from "src/products/queries/getInfiniteProducts"

const Page = function ({ products }) {
  return (
    <div>
      <button
        onClick={async () => {
          const queryKey = getInfiniteQueryKey(getInfiniteProducts)
          getQueryClient().invalidateQueries(queryKey)
        }}
      >
        Invalidate
      </button>
    </div>
  )
}
export default Page
```

## API {#api-2}

```js
const queryKey = getQueryKey(resolver, inputArgs?)
```

### 引数 {#arguments-2}

- `resolver:` Blitzの[クエリリゾルバ](./query-resolvers) または [ミューテーションリゾルバ](./mutation-resolvers)
  - **必須**
- `inputArgs`
  - 任意
  - `resolver`に渡される引数
  - これを提供しない場合、すべての`resolver`クエリの基本クエリキーを返します
  - これを提供すると、`resolver`と`inputArgs`の組み合わせの特定のクエリキーを返します

### 戻り値 {#returns-2}

- `results`
  - リゾルバから返される正確な結果

## `setQueryData` {#set-query-data}

これの動作は[useQueryにより返されるsetQueryData](./use-query#setQueryData)と同じです。違いは、useQueryにバインドされておらず、`resolver`と`inputArgs`を期待していることです。また、クエリを無効にして再フェッチを引き起こします。クエリ無効化をオフにするには、オプションオブジェクト`{refetch: false}`を`opts`引数として渡します。

### 例 {#example-4}

```tsx
import { setQueryData } from "@blitzjs/rpc"

export default function (props: { query: { id: number } }) {
  const [product] = useQuery(getProduct, {
    where: { id: props.query.id },
  })
  const [updateProjectMutation] = useMutation(updateProject)

  return (
    <Formik
      initialValues={product}
      onSubmit={async (values) => {
        try {
          const product = await updateProjectMutation(values)
          setQueryData(
            getProduct,
            { where: { id: props.query.id } },
            product
          )
        } catch (error) {
          alert("Error saving product")
        }
      }}
    >
      {/* ... */}
    </Formik>
  )
}
```

## API {#api-3}

```js
const promise = setQueryData(resolver, inputArgs, newData, opts?)
```

### 引数 {#arguments-3}

- `resolver:` Blitzの[クエリリゾルバ](./query-resolvers)。
  - **必須**
- `inputArgs`
  - **必須**
  - `resolver`に渡される引数
  - 指定された`resolver`と`inputArgs`の組み合わせのデータを設定します
- `newData`
  - **必須**
  - 関数でない値を渡すと、データはこの値に更新されます
  - 関数を渡すと、古いデータ値を受け取り、新しいものを返すことが期待されます。
- `opts`
  - 任意
  - 追加のオプションが含まれるオブジェクト。
  - オプションオブジェクト`{refetch: false}`を渡して再フェッチを無効にします。

### 戻り値 {#returns-3}

- `results`
  - 新しいデータが再フェッチされ終わった時に解決されるプロミスを返します。