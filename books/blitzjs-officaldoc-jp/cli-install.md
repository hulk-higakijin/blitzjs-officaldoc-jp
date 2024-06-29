    ---
title: blitz インストール
sidebar_label: blitz インストール
---

**エイリアス: `blitz i`**

このコマンドを使用して、Blitzレシピをプロジェクトにインストールします。

公式レシピおよびカスタムのサードパーティ製レシピの両方に対応しています。レシピの名前は以下の形式で指定できます：

- `blitz install` 公式Blitzレシピのリストから選択してインストール

- `blitz install official-recipe` Blitzチームの公式レシピ
- `blitz install ./recipes/my-local-recipes.ts` ローカルで作成したレシピ
- `blitz install github-user/my-published-recipe` GitHubでホストされているカスタムレシピ

また、レシピ自体へ引数を`key=value`の形式で渡すこともできます（受け取る場合）：

```bash
$ blitz install my-recipe key=value
```

レシピに関する詳細情報は、公式ドキュメントの[レシピのインストール](using-recipes)および[自分で書く](writing-recipes)を参照してください。

| 引数   | 必須 | 説明                                                                                     |
| ------ | ---- | ---------------------------------------------------------------------------------------- |
| `name` | いいえ | インストールするレシピの名前、ローカルレシピへのパス、またはGitHubリポジトリ名  |

#### オプション

| 引数      | 短縮 | 説明                                                                                   | デフォルト |
| --------- | ---- | -------------------------------------------------------------------------------------- | ---------- |
| `--yes`   | `-y` | ユーザーの確認なしにレシピを自動的にインストールします。                                | `false`    |
| `--env`   | `-e` | アプリの環境名を設定します。[詳細はこちら](/docs/custom-environments#custom-environments). | なし       |

#### 出力例

```bash
> blitz install
? インストールするレシピを選択してください …
❯ base-web
  bumbag-ui
  chakra-ui
  emotion
### など
```

```bash
> blitz install tailwind

✅ 2つの依存関係がインストールされました
✅ postcss.config.js、tailwind.config.jsが正常に作成されました
✅ app/styles/button.css、app/styles/index.cssが正常に作成されました
✅ 1つのファイルが変更されました: app/pages/_app.tsx

🎉 Tailwind CSSのレシピが正常に完了しました！機能はBlitzアプリで完全に構成されています。
```