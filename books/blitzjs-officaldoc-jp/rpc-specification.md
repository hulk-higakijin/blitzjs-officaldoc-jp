---
id: rpc-specification
title: RPC仕様
sidebar_label: RPC仕様
---

:::alert
これはBlitzのクエリとミューテーションを非Blitzアプリケーションで使用するためのドキュメントです。例えば、モバイルアプリケーションや他のアプリケーションから使用する場合です。

**アプリケーションでクエリとミューテーションを使用するには、[クエリの使用方法](./query-usage)と[ミューテーションの使用方法](./mutation-usage)を参照してください。**

**アプリケーションに外部APIとしてアクセスしたい場合は、[API Routes](https://nextjs.org/docs/api-routes/introduction)を参照してください。**
:::

## リクエストの詳細 {#request-details}

HTTPの観点から見ると、クエリとミューテーションの間に違いはありません。どちらもPOSTリクエストに変換される関数呼び出しです。

## HEAD {#head}

ラムダをウォームアップします。例えば、クライアントでミューテーションを使用するコンポーネントがレンダリングされるとき、ページの読み込み時にすぐにミューテーションエンドポイントにHEADリクエストを発行して、実際のミューテーションリクエストがウォームなラムダにヒットする可能性を高めます。

常にHTTP `200 OK` を返す必要があります。

### HEADレスポンス {#head-responses}

```http
HTTP/1.1 200 OK
```

## POST {#post}

クエリまたはミューテーションを実行します。

リクエストボディは以下のキーを含むJSONでなければなりません：

- params (必須) - これはクエリやミューテーションの第一引数として提供されます。
- version (オプション) - ビルドされたRPCクライアントのバージョンを含む文字列

レスポンスボディは以下のキーを含むJSONです：

- result - クエリ/ミューテーション関数から返された正確な結果、またはエラーが発生した場合はnull
- error - 成功時はnull、それ以外の場合はオブジェクト

**POSTボディの例:**

```json
{
  "params": {
    "where": {
      "id": 1
    }
  }
}
```

### POSTレスポンス {#post-responses}

#### 成功

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "result": {
    "name": "Hello World",
    "description": "This is awesome :)"
  },
  "error": null
}
```

#### 不正なリクエスト (JSONでない場合)

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "result": null,
  "error": {
    "message": "Request body is not valid JSON"
  }
}
```

#### 不正なリクエスト (paramsが不足している場合)

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "result": null,
  "error": {
    "message": "Request body is missing the 'params' key"
  }
}
```

#### 見つからない (間違ったメソッド)

```http
HTTP/1.1 404 Not Found
```

## バージョニング {#versioning}

クライアントとサーバーの通信がある場合、クライアントとサーバーのコードが同期しなくなるリスクが常に存在し、その結果エラーが発生する可能性があります。例えば、単一のオブジェクトを入力とするミューテーション関数があると仮定します。このミューテーションの入力を単一の配列に変更するとします。変更を行い、デプロイします。サーバーコードは更新されましたが、ユーザー全員がブラウザで古いクライアントコードを実行しているままです。つまり、クライアントコードがオブジェクトをミューテーションに送信し、配列を期待しているためエラーが発生する可能性があります。

1つの解決策は、後方互換性のある変更だけを行うことですが、これができない場合もあります。

もう一つの解決策は、クライアントコードとサーバーコードのバージョンを追跡することです。バージョンの不一致があれば、ユーザーにウェブページを再読み込みするように促すフレンドリーメッセージを表示することができます。

```js
// *: ユーザーにフレンドリーメッセージを表示してRPC呼び出しを遮蔽する例
```
