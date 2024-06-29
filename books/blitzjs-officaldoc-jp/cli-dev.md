---
title: blitz dev
sidebar_label: blitz dev
---

**エイリアス: `blitz d`**

Blitz開発サーバーを起動します。

これは環境変数の読み込みを改善するためのNext CLIの非常に薄いラッパーです。

```bash
blitz dev -e staging
```

これは次のことを行います:

1. 指定した`-e X`フラグに基づいて、適切な `.env.X` および `.env.X.local` ファイルを読み込みます。例: `-e staging`は `.env.staging` を読み込みます。
2. `process.env.APP_ENV` を環境名に設定します。例: `-e staging` => `APP_ENV=staging`。
3. 他のすべての引数を次のビルドコマンドに直接渡します。

#### オプション

| オプション                 | 省略形   | 説明                                                                                     | デフォルト       |
| -------------------------- | -------- | ---------------------------------------------------------------------------------------- | ---------------- |
| `--hostname`               | `-H`     | サーバーに使用するホスト名を設定します。                                                | `"localhost"`  |
| `--port`                   | `-p`     | サーバーがリッスンするポートを設定します。                                              | `3000`          |
| `--no-incremental-build`   |          | インクリメンタルビルドを無効にし、新しいキャッシュから開始します。                      | `false`         |
| `--inspect`                |          | Node.jsインスペクターを有効にします。                                                     | `false`         |
| `--env`                    | `-e`     | アプリの環境名を設定します。 [詳細はこちらをお読みください](/docs/custom-environments#custom-environments)。 | None            |

#### 例

```bash
blitz dev
blitz dev -p 4000
```
