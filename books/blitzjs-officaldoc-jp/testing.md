    ---
title: テスト
sidebar_label: テスト
---

`blitz` 依存関係は `vitest`、`@testing-library/react`、`@testing-library/react-hooks` も自動的にインストールおよび設定します。

## Vitest {#vitest}

[Vitest](https://vitest.dev/) は使いやすいテストフレームワークです。アプリでテストを実行するには、ターミナルから `vitest run` または `vitest` を実行します。

新しいテストを作成するには、`something.test.ts`（または `.js`、`.tsx` など）という名前のファイルを作成します。以下は例です：

```ts
import something from "./somewhere"

it("does something", () => {
  const result = something()
  expect(result).toBe(true)
})
```

> Vitestの利用に関する詳細情報については、[Vitestのドキュメント](https://vitest.dev/api/#tomatch)を読んでください。

### Vitest 設定 {#vitest-config}

[vitestの設定をカスタマイズする必要がある場合](https://vitest.dev/config/)、`vitest.config.ts` にて行えます。

```ts
// vitest.config.ts
import { loadEnvConfig } from "@next/env"
import { defineConfig } from "vitest/config"

import react from "@vitejs/plugin-react"
import tsconfigPaths from "vite-tsconfig-paths"

const projectDir = process.cwd()
loadEnvConfig(projectDir)

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    dir: "./",
    globals: true,
    setupFiles: "./test/setup.ts",
    coverage: {
      reporter: ["text", "json", "html"],
    },
  },
})
```

### クライアント vs サーバー環境 {#client-vs-server-environments}

以下のテストファイルはサーバー環境で実行されます：

- `api`、`queries`、または `mutations` フォルダ内の `*.test.*` ファイル
- `*.server.test.*` ファイル

その他のすべてのファイルはクライアント環境（jsdom を使用）で実行されます。

## @testing-library {#testing-library}

[@testing-library](https://testing-library.com/docs/) は、ユーザー中心の方法でUIコンポーネントをテストするためのパッケージ群です。過去に Enzyme を使用したことがある場合、それに代わるものとして良い選択肢です。

以下のパッケージがインストールされています。必要に応じてドキュメントを参照してください。

- [@testing-library/dom](https://testing-library.com/docs/dom-testing-library/intro)
- [@testing-library/jest-dom](https://testing-library.com/docs/ecosystem-jest-dom)
- [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro)
- [@testing-library/react-hooks](https://react-hooks-testing-library.com/)

## テストユーティリティ {#testing-utilities}

Blitz は生成された `test/utils.tsx` ファイル内に基本的なテストユーティリティを提供します。これらは Vitest と React Testing Library を使用して、テスト環境の高度な設定ができるようにするためのものです。このファイルをカスタマイズすると、グローバルプロバイダーや Blitz が内部で処理するルーティングやデータクエリなどの特定の部分をテストするのに便利です。

### `BlitzProvider` {#blitz-provider}

このコンポーネントは、テスト内でのデータクエリやミューテーションのためのグローバルプロバイダーとして機能します。内部的には `react-query` の `QueryClient` コンポーネントをラップし、追加の設定オプションを提供します。

`BlitzProvider` は以下のプロップを受け取ります：

- `client` - React Query によって使用されるグローバルクエリクライアント。デフォルトでは、`@blitzjs/rpc` の `queryClient`（`getQueryClient` 関数を通じてアクセス可能） が使用されますが、オーバーライドすることもできます。
- `contextSharing` - プロバイダーコンテクストが複数のアプリケーション間で共有できるかどうかを制御します。マイクロフロントエンドなどの特定の環境で作業する際に便利です。デフォルトは `false` です。
- `dehydratedState` - サーバー側のクエリ結果を含むサーバー状態をアプリケーションに渡します。プリレンダリング（事前描画）されたクエリをテストするのに便利です。

## Cypress を用いたエンドツーエンドテスト {#e2e-cypress}

[Cypress](https://www.cypress.io/) は単体テストと統合テストの両方のオプションを提供するフロントエンドテストフレームワークです。

Blitz アプリケーションに統合する計画がある場合、[Cypress 設定の例](https://github.com/blitz-js/blitz/tree/canary/examples/cypress)を参照してください。この例にはシンプルなファクトリーパターン、ユーザーを作成してログインするためのコマンド、およびいくつかのテスト例が含まれています。
