```mdx
---
title: blitz ルート
sidebar_label: blitz ルート
---

**エイリアス: `blitz r`**

全てのBlitz URLルートを表示します

#### 例

```bash
> blitz routes

✔ コンパイル完了

┌──────┬─────────────────────────────────────┬───────────────────────────────────┬──────┐
│ HTTP │ ソースファイル                     │ URI                               │ タイプ │
├──────┼─────────────────────────────────────┼───────────────────────────────────┼──────┤
│ GET  │ app/pages/index.tsx                 │ /                                 │ ページ │
│ GET  │ app/pages/ssr.tsx                   │ /ssr                              │ ページ │
│ GET  │ app/auth/pages/login.tsx            │ /login                            │ ページ │
│ GET  │ app/auth/pages/signup.tsx           │ /signup                           │ ページ │
│ GET  │ app/users/pages/users/index.tsx     │ /users                            │ ページ │
│  *   │ app/api/auth/[...auth].ts           │ /api/auth/[...auth]               │ API  │
│ POST │ app/auth/mutations/login.ts         │ /api/auth/mutations/login         │ RPC  │
│ POST │ app/auth/mutations/logout.ts        │ /api/auth/mutations/logout        │ RPC  │
│ POST │ app/auth/mutations/signup.ts        │ /api/auth/mutations/signup        │ RPC  │
│ POST │ app/users/mutations/trackView.ts    │ /api/users/mutations/trackView    │ RPC  │
│ POST │ app/users/queries/getCurrentUser.ts │ /api/users/queries/getCurrentUser │ RPC  │
│ POST │ app/users/queries/getUser.ts        │ /api/users/queries/getUser        │ RPC  │
│ POST │ app/users/queries/getUsers.ts       │ /api/users/queries/getUsers       │ RPC  │
└──────┴─────────────────────────────────────┴───────────────────────────────────┴──────┘
✨ 4.60秒で完了
```
```