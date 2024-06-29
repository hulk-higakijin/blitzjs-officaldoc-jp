---
title: カスタム環境変数
sidebar_label: カスタム環境
---

## カスタム環境の利用 {#custom-environments}

Next.jsのデフォルト環境には `production`、`development`、`test` の3つがあります。ただし、時には `staging` のような追加の環境が必要になることがあります。このような環境のために環境変数を別のファイルに保存したい場合、`-e`（`--env`）フラグを渡すか、`APP_ENV`を直接設定することでアプリ環境名を提供できます。例えば、次のようになります。

```
APP_ENV=staging blitz build    // .env.stagingを読み込む

blitz prisma generate -e staging  // .env.stagingを読み込む

blitz dev --env staging  // .env.stagingを読み込む

// 開発モードで本番環境の変数が誤って読み込まれる問題も解決されます
blitz dev --e production
```

これにより、追加の環境ファイル `.env.<APP_ENV>` と `.env.<APP_ENV>.local` が読み込まれます。

複数の環境ファイルを読み込む必要がある場合は、すべての環境をカンマで区切った文字列として渡すことができます。例えば次のようになります。

```bash
blitz dev --env staging,production
# .env.staging.localを読み込みました
# .env.stagingを読み込みました
# .env.production.localを読み込みました
# .env.productionを読み込みました
# .env.localを読み込みました
# .envを読み込みました
```
