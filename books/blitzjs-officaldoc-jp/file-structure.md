---
id: file-structure
title: ファイル構造
sidebar_label: ファイル構造
---

#### 指導原則

1. 一緒に変更するファイルは一緒に配置するべきです。
2. 最小限の要件と最大限の柔軟性

#### ファイル構造の例

```
├── src/
│   ├── blitz-client.ts
│   ├── blitz-server.ts
│   ├── core/
│   │   ├── components/
│   │   │   ├── Form.tsx
│   │   │   └── LabeledTextField.tsx
│   │   └── layouts/
│   │       └── Layout.tsx
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── SignupForm.tsx
│   │   ├── mutations/
│   │   │   ├── login.ts
│   │   │   ├── logout.ts
│   │   │   └── signup.ts
│   │   ├── auth-utils.ts
│   │   └── validations.ts
│   ├── users/
│   │   ├── hooks/
│   │   │   └── useCurrentUser.ts
│   │   └── queries/
│   │       └── getCurrentUser.ts
│   ├── projects/
│   │   ├── components/
│   │   │   ├── Project.tsx
│   │   │   ├── ProjectForm.tsx
│   │   │   └── Projects.tsx
│   │   ├── mutations/
│   │   │   ├── createProject.ts
│   │   │   ├── createProject.test.ts
│   │   │   ├── deleteProject.ts
│   │   │   ├── deleteProject.test.ts
│   │   │   ├── updateProject.ts
│   │   │   └── updateProject.test.ts
│   │   └── queries/
│   │       ├── getProject.ts
│   │       └── getProjects.ts
│   └── pages/
│       ├── api/
│       │   └── stripe-webhook.ts
│       ├── 404.tsx
│       ├── _app.tsx
│       ├── _document.tsx
│       ├── index.test.tsx
│       ├── index.tsx
│       ├── auth/
│       │   ├── login.tsx
│       │   └── signup.tsx
│       └── projects/
│           ├── [id]/
│           │   └── edit.tsx
│           ├── [id].tsx
│           ├── index.tsx
│           └── new.tsx
├── db/
│   ├── index.ts
│   ├── schema.prisma
│   └── seeds.ts
├── integrations/
│   └── sentry.ts
├── public/
│   ├── favicon.ico*
│   └── logo.png
├── test/
│   ├── setup.ts
│   └── utils.tsx
├── README.md
├── next.config.js
├── vitest.config.ts
├── package.json
├── tsconfig.json
├── types.ts
└── yarn.lock
```

#### `src`

コアアプリケーションコードが含まれます。

- `src/`内にネストされたファイル構造は自由に設定できますが、以下をお勧めします：
  - 第一層のフォルダは `core/`、`projects/`、`users/`、`billing/` などの「ドメインコンテキスト」にします。
  - `components/` や `hooks/` などの他のフォルダは上記のドメインコンテキストフォルダ内に配置します。
- `queries/` および `mutations/` は Blitz のクエリおよびミューテーション用です。
  各クエリおよびミューテーションは、ファイルパスに対応する URL に公開されます。
- `blitz-client.ts` と `blitz-server.ts` には Blitz Toolkit 設定が含まれます。
- `pages` は Next.js の `pages` フォルダーと同じセマンティクスを持ちます。ここにあるすべてのファイルは、ファイルパスに対応する URL にマッピングされます。

#### `db`

データベースの構成、スキーマ、およびマイグレーションファイルが含まれます。
`db/index.js` は初期化済みのデータベースクライアントをエクスポートし、アプリ内のどこでも使用できるようにします。

#### `integrations`

サードパーティ統合コードが含まれます。例：`auth0.js`、`twilio.js` など。
これらのファイルは、共有構成でクライアントをインスタンス化するのに適した場所です。

#### `api`

フォルダ `src/pages/api` 内の任意のファイルは `/api/*` にマッピングされ、ページではなく API エンドポイントとして扱われます。
これらはサーバーサイド専用のバンドルであり、クライアントサイドのバンドルサイズを増やすことはありません。

#### `public`

ここにあるファイルはすべて、アプリのルート URL から静的に提供されます。

### その他の注意事項 {#other-notes}

- 全てのトップレベルフォルダは自動的にエイリアスされます。例えば、アプリ内のどこからでも `src/projects/queries/getProject` からインポートできます。
