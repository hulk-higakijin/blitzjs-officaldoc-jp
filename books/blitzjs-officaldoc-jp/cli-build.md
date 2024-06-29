    ---
title: blitz build
sidebar_label: blitz build
---

**別名: `blitz b`**

Blitzアプリの本番ビルドを作成します。これにより、BlitzアプリがNext.jsランタイムコードにコンパイルされ、`.next`ディレクトリ内に配置されます。

これは環境変数の読み込みを改善するためのNext CLIの非常に薄いラッパーです。

```bash
blitz dev -e staging
```

以下の操作を実行します:

1. 指定した`-e X`フラグに基づいて、正しい`.env.X`および`.env.X.local`ファイルを読み込みます。例: `-e staging`は`.env.staging`を読み込みます。
2. `process.env.APP_ENV`を環境名に設定します。例: `-e staging` => `APP_ENV=staging`。
3. 他のすべての引数を次のビルドコマンドに直接渡します。

#### オプション

| オプション  | ショートハンド | 説明                                                                                     | デフォルト |
| ----------- | -------------- | ---------------------------------------------------------------------------------------- | ---------- |
| `--env`     | `-e`           | アプリケーション環境名を設定します。 [詳細はこちら](/docs/custom-environments#custom-environments)。 | なし       |

#### 例

```bash
blitz build
```