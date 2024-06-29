```mdx
---
title: useMutation
sidebar_label: useMutation
---

### 例 {#example}

```tsx
import {useMutation} from "@blitzjs/rpc"
import updateProject from 'src/projects/mutations/updateProject'

function (props) {
  const [updateProjectMutation] = useMutation(updateProject)

  return (
    <Formik
      onSubmit={async values => {
        try {
          const project = await updateProjectMutation(values)
        } catch (error) {
          alert('プロジェクトの保存中にエラーが発生しました')
        }
      }}>
      {/* ... */}
    </Formik>
  )
}
```

<br />

> 詳細な例や使用例については
> [mutation 使用法のドキュメント](./mutation-usage)を参照してください。

## API {#api}

```js
const [
  invoke,
  {
    data,
    error,
    isError,
    isIdle,
    isLoading,
    isPaused,
    isSuccess,
    mutate,
    reset,
    status,
  },
] = useMutation(mutationResolver, {
  mutationKey,
  onError,
  onMutate,
  onSettled,
  onSuccess,
  useErrorBoundary,
})

const promise = invoke(inputArguments, {
  onError,
  onSettled,
  onSuccess,
})
```

### 引数 {#arguments}

- `mutationResolver:` [Blitz ミューテーションリゾルバー](./mutation-resolvers)
  - **必須**
- `options`
  - 任意

### オプション {#options}

- `mutationKey: string`
  - 任意
  - `queryClient.setMutationDefaults`で設定されたデフォルトを継承するため、または開発ツールでミューテーションを識別するためにミューテーションキーを設定できます。
- `onMutate: (variables: TVariables) => Promise<TContext | void> | TContext | void`
  - 任意
  - この関数はミューテーション関数が実行される前に実行され、ミューテーション関数が受け取るのと同じ変数が渡されます。
  - ミューテーションが成功することを期待してリソースの楽観的な更新を行うのに便利です。
  - この関数から返される値は、ミューテーションの失敗時に`onError`および`onSettled`関数に渡され、楽観的な更新を取り消すのに役立つことがあります。
- `onSuccess: (data: TData, variables: TVariables, context?: TContext) => Promise<void> | void`
  - 任意
  - ミューテーションが成功したときにこの関数が実行され、ミューテーションの結果が渡されます。
  - もしプロミスを返す場合、それは解決されるまで待機されます。
- `onError: (err: TError, variables: TVariables, context?: TContext) => Promise<void> | void`
  - 任意
  - ミューテーションでエラーが発生したときにこの関数が実行され、エラーが渡されます。
  - もしプロミスを返す場合、それは解決されるまで待機されます。
- `onSettled: (data: TData, error: TError, variables: TVariables, context?: TContext) => Promise<void> | void`
  - 任意
  - この関数はミューテーションが成功した場合でもエラーが発生した場合でも実行され、データまたはエラーが渡されます。
  - もしプロミスを返す場合、それは解決されるまで待機されます。
- `retry: boolean | number | (failureCount: number, error: TError) => boolean`
  - `false`の場合、失敗したミューテーションはデフォルトでは再試行されません。
  - `true`の場合、失敗したミューテーションは無限に再試行されます。
  - `3`のような`number`値の場合、失敗したミューテーションはその数に達するまで再試行されます。
- `retryDelay: number | (retryAttempt: number, error: TError) => number`
  - この関数は`retryAttempt`整数と実際のエラーを受け取り、次回の試行までの遅延をミリ秒単位で返します。
  - `attempt => Math.min(attempt > 1 ? 2 ** attempt * 1000 : 1000, 30 * 1000)`のような関数は指数関数的なバックオフを適用します。
  - `attempt => attempt * 1000`のような関数は線形バックオフを適用します。
- `useErrorBoundary: boolean`
  - 任意
  - デフォルトではグローバルクエリ構成の`useErrorBoundary`値は`false`です。
  - ミューテーションエラーがレンダリングフェーズでスローされ、最も近いエラーバウンダリーに伝播されるようにしたい場合は、これをtrueに設定します。

### 戻り値 {#returns}

`[mutate, mutationExtras]`

##### `mutate: (variables: TVariables, { onSuccess, onSettled, onError }) => Promise<TData>`

- ミューテーション関数で変数を渡してミューテーションをトリガーし、`useMutation`に渡されたオプションをオーバーライドするためのオプションです。
- `variables: TVariables`
  - 任意
  - `mutationFn`に渡す変数オブジェクトです。
- 残りのオプションは`useMutation`フックで説明されたオプションを拡張します。
- 複数のリクエストを行う場合、最新のリクエストのみが`onSuccess`を実行します。

##### `mutationExtras: オブジェクト`

- `status: string`
  - 状態は以下のようになります。
    - `idle` ミューテーション関数が実行される前の初期状態です。
    - `loading` ミューテーションが現在実行中の場合。
    - `error` 最後のミューテーション試行がエラーになった場合。
    - `success` 最後のミューテーション試行が成功した場合。
- `isIdle`, `isLoading`, `isSuccess`, `isError`: `status`から派生したブール値
- `data: undefined | unknown`
  - デフォルトは`undefined`
  - 最後に正常に解決されたデータです。
- `error: null | TError`
  - エラーが発生した場合のクエリエラーオブジェクト。
- `reset: () => void`
  - ミューテーションの内部状態をクリアする関数（つまり、ミューテーションを初期状態にリセットします）。
```