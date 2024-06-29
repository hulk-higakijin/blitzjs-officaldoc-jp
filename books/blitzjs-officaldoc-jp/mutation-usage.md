---
title: ミューテーションの使用
---

## React コンポーネントで {#in-a-react-component}

[Blitz ミューテーション](./mutation-resolvers)の1つを使用するには、ミューテーションリゾルバーを使って`useMutation`フックを呼び出します。

これはタプル配列 `[invoke, mutationExtras]` を返し、`invoke` はイベントハンドラまたはエフェクト内で直接呼び出すことができる非同期関数です。

`useMutation` は[`react-query`](https://github.com/tannerlinsley/react-query)の `useMutation` フックの上に薄くレイヤーを設けただけのものです。

```tsx
import {useMutation} from '@blitzjs/rpc'
import updateProject from 'app/projects/mutations/updateProject'

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

詳細については、[`useMutation` ドキュメント](./use-mutation)を参照してください。

<Card type="info" title="🤔">

サーバーコードをコンポーネントにインポートしているのにどうして動くのか疑問に思うかもしれませんが：ビルド時には、直接関数のインポートはネットワークコールに置き換えられます。クエリ関数のコードはクライアントコードに含まれません。

</Card>

## エラーハンドリング {#error-handling}

サーバーコードからスローされたエラーは、クライアントでも予想通りにスローされます。そのため、`invoke` を `try {} catch {}` または `.catch()` でラップすることができます。

## キャッシュ無効化 {#cache-invalidation}

#### `setQueryData()`

- [`useQuery`](./use-query) はキャッシュを手動で更新するために使用できる `setQueryData` 関数を返します
- `setQueryData` を呼び出すと、自動的に初期のクエリの再フェッチがトリガーされ、正しいデータが得られるようにします。オプションオブジェクト `{refetch: false}` を第2引数として渡すことで、再フェッチを無効にします。

```tsx
export default function (props: { query: { id: number } }) {
  const [product, { setQueryData }] = useQuery(getProduct, {
    where: { id: props.query.id },
  })
  const [updateProjectMutation] = useMutation(updateProject)

  return (
    <Formik
      initialValues={product}
      onSubmit={async (values) => {
        try {
          const product = await updateProjectMutation(values)
          setQueryData(product)
        } catch (error) {
          alert("製品の保存中にエラーが発生しました")
        }
      }}
    >
      {/* ... */}
    </Formik>
  )
}
```

#### `refetch()`

[`useQuery`](./use-query) はクエリの再読み込みをトリガーするために使用できる `refetch` 関数を返します。

```tsx
export default function (props) {
  const [product, { refetch }] = useQuery(getProduct, {
    where: { id: props.id },
  })
  const [updateProjectMutation] = useMutation(updateProject)

  return (
    <Formik
      initialValues={product}
      onSubmit={async (values) => {
        try {
          const product = await updateProjectMutation(values)
          refetch()
        } catch (error) {
          alert("製品の保存中にエラーが発生しました")
        }
      }}
    >
      {/* ... */}
    </Formik>
  )
}
```

キーワード: 更新, 再検証, 無効化

#### `invalidateQuery()`

別のコンポーネント内でクエリを無効化する必要がある場合のもう一つの方法は、`blitz` によって提供される[`invalidateQuery()`](./resolver-utilities#invalidateQuery)関数を使用することです。

##### 例

```tsx
import { invalidateQuery } from "@blitzjs/rpc" // ハイライト行
import getProducts from "app/products/queries/getProducts"

const Page = function ({ products }) {
  return (
    <div>
      <button
        onClick={() => {
          invalidateQuery(getProducts) // ハイライト行
        }}
      >
        無効化
      </button>
    </div>
  )
}
export default Page
```

#### 自動的に

クエリキャッシュを自動的に無効にする方法を検討しています。まだ方法は確立されていませんが、[オープンイシューに貢献することができます！](https://github.com/blitz-js/blitz/issues/586)

## 楽観的更新 {#optimistic-updates}

楽観的なUIパターンは、ユーザーの操作に対するアプリの応答性を向上させるための技術です。ユーザーがアクションを実行すると (例：投稿に「いいね」をする)、その操作が成功したと仮定してアクティビティスピナーや読み込みアニメーションを表示せずに済みます。ローカルキャッシュとインターフェースをすぐに更新し、成功した結果を表示します。

実際のレスポンスがアクションから届いたときに、データベースから返された真の値を反映するようにUIを更新します。ほとんどの場合、リクエストが成功すればユーザーには見えないまま変更が即座に行われたように見えます。

エラーが発生した場合には、ローカルの変更を元に戻し、データを再フェッチして、必要に応じてエラーメッセージを表示します。

こちらは、`updateProduct` ミューテーションを呼び出して編集を保存する `onSubmit` 関数を持つシンプルなフォームコンポーネントです。

```tsx
export default function (props: { query: { id: number } }) {
  const [product, { setQueryData }] = useQuery(getProduct, {
    where: { id: props.query.id },
  })
  const [updateProjectMutation] = useMutation(updateProject)

  return (
    <Formik
      initialValues={product}
      onSubmit={async (values) => {
        try {
          const product = await updateProjectMutation(values)
          setQueryData(product)
        } catch (error) {
          alert("製品の保存中にエラーが発生しました")
        }
      }}
    >
      {/* ... */}
    </Formik>
  )
}
```

これを拡張して、ユーザーに対して即座のフィードバックをサポートし、編集後の製品情報を即座に表示するようにできます。`getProducts` クエリのローカルキャッシュを、データベースに送信しようとしている変更で更新します。レスポンスを待っている間、{refetch: false} を `setQueryData()` 関数の第2引数に渡して、ミューテーションが改めて古いデータの再フェッチをトリガーするのを防ぎます。

```tsx
export default function (props: { query: { id: number } }) {
  const [product, { setQueryData }] = useQuery(getProduct, {
    where: { id: props.query.id },
  })
  const [updateProjectMutation] = useMutation(updateProject)

  return (
    <Formik
      initialValues={product}
      onSubmit={async (values) => {
        try {
          // フォームの値でローカルキャッシュを更新し、リフェッチをトリガーしない
          setQueryData(values, { refetch: false }) // ハイライト行
          const product = await updateProjectMutation(values)
          // 結果でローカルキャッシュを更新し、その後リフェッチ
          setQueryData(product)
        } catch (error) {
          // 元の結果でキャッシュを戻し、その後リフェッチ
          setQueryData(product) // ハイライト行
          alert("製品の保存中にエラーが発生しました")
        }
      }}
    >
      {/* ... */}
    </Formik>
  )
}
```
