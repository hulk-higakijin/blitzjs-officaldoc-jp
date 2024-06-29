---
title: レシピの使用
---

Blitz Recipesを使用すると、単一のコマンドでサードパーティのコードをインストールおよび構成できます！

**[公式レシピのリストを見る](https://github.com/blitz-js/blitz/tree/canary/recipes).**

- ✅ ワンコマンド: `blitz install [レシピ名]`
- ✅ 自分で作成する [カスタムレシピ](writing-recipes)

Blitz Recipesは、npm依存関係やその他のコードをアプリケーションに追加するための事前定義されたフォーミュラです。私たちは
[Gatsby Recipes](https://www.gatsbyjs.org/docs/recipes/)からインスピレーションを受け（そして私たち自身の
実装の構想段階で彼らの助けに非常に感謝しています）、既存のサードパーティ
依存関係をアプリケーションに追加するフローを改善しました。

サードパーティのコードをインストールするのは、設定に多くのステップが必要で非常に複雑になることがよくあります。
Blitz Recipesはその余分なステップを減らすのに役立ちます。公式のNext.jsの例に精通している場合、レシピはその代替手段です。
サードパーティライブラリを統合するには、統合の例を調べなくても、1つのコマンドを実行するだけです。

```bash
> blitz install tailwind

✅ 2つの依存関係がインストールされました
✅ postcss.config.js、tailwind.config.jsが正常に作成されました
✅ app/styles/button.css、app/styles/index.cssが正常に作成されました
✅ 1ファイルが変更されました: app/pages/_app.tsx

🎉 Tailwind CSSのレシピが正常に完了しました！その機能はBlitzアプリで完全に構成されています。
```

### レシピのインストール方法 {#how-to-install-a-recipe}

Blitz Recipesの完全なリストは
[GitHubリポジトリ](https://github.com/blitz-js/blitz/tree/canary/recipes)にあります。
レシピの名前はコードを含むディレクトリ名です。インストールしたいレシピの名前を取得し、
[`blitz install`](cli-install) に渡すと、インストーラーがプロセスを案内し、変更の説明を提供してくれます。たとえば、今日のレシピディレクトリには
`tailwind` のレシピがあり、[Tailwind CSS](https://tailwindcss.com/)をインストールします。
`blitz install tailwind`を実行することでセットアップが完了し、残りはBlitzに任せることができます。

レシピの最良の部分の1つは、バージョン管理されておらず、直接GitHubから取得されることです。レシピに変更を加えてGitHubで
マージすると、すぐに実行可能になります。同様に、[カスタムレシピ](writing-recipes)も同じです。
