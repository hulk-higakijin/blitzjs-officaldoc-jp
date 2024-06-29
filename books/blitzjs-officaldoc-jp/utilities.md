---
title: ユーティリティ
---

Blitzは便利なユーティリティ関数を提供します

## `validateZodSchema` {#validate-zod-schema}

このユーティリティ関数は、[`zod`](https://github.com/colinhacks/zod) スキーマを使用して入力を検証し、エラーをフォーマットしてフォームライブラリで使用できるようにします。

これは現在、React Final FormとFormik、および同じAPIを持つ他のライブラリと互換性があります。

### 例 {#validate-zod-schema-example}

```tsx
import {validateZodSchema} from 'blitz'

<FinalForm
  initialValues={initialValues}
  validate={validateZodSchema(schema)}
  ...
```

### API {#validate-zod-schema-api}

```js
const validationFunction = validateZodSchema(MyZodSchema)
```

#### 引数

- `ZodSchema:` [zod](https://github.com/colinhacks/zod) スキーマ
  - **必須**
- `parserType:` 同期解析を示す文字列 "sync" または非同期解析を示す文字列 "async" のいずれかをこのパラメーターに渡します。省略すると、デフォルトで非同期になります。
  - **任意**

#### 返り値

フォームコンポーネントの`validate`プロップに渡す検証関数を返します。いくつかの値を受け取り、エラーを含むPromiseまたはオブジェクトを返します。

```
(values: any, parserType?: "sync" | "async") =>  Promise<Object> | Object
```

> **注**: "sync"または"async"のいずれかの変数を渡すことは現在サポートされていません。オプションのparserTypeパラメータを使用する場合は、明示的に"sync"または"async"を渡す必要があります。
>
> この機能に関する強力なユースケースがある場合は、Githubにissueをオープンしてください。

## `formatZodError` {#format-zod-error}

このユーティリティ関数は、ZodErrorを受け取り、フォームライブラリで使用できるようにフォーマットします。

これは現在、React Final FormとFormik、および同じAPIを持つ他のライブラリと互換性があります。

### 例 {#format-zod-error-example}

```tsx
import {formatZodError} from 'blitz'

<FinalForm
  initialValues={initialValues}
  validate={values => {
    try {
      schema.parse(values)
    } catch (error) {
      return formatZodError(error)
    }
  }}
  ...
```

### API {#format-zod-error-api}

```js
const formattedErrorsObject = formatZodError(myZodError)
```

#### 引数

- `ZodError:` [zod](https://github.com/colinhacks/zod) エラー
  - **必須**

#### 返り値

元のスキーマと同じ形状を取るエラーオブジェクト
