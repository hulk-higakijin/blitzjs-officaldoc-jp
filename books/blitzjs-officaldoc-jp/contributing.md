---
title: How to Contribute
---

👋 Blitzに貢献することに興味を持ってくださり、とても嬉しいです！以前にオープンソースの経験がなくても、スタートをサポートします :)

## まず最初に {#first-things-first}

1. オープンソース初心者ですか？このガイドを見てください
   [How to Contribute to an Open Source Project on GitHub](https://egghead.io/courses/how-to-contribute-to-an-open-source-project-on-github)
2. [Blitzの行動規範](./code-of-conduct)に慣れ親しんでください
3. [コミュニティの運営方針](./how-the-community-operates)を学びましょう

## 何に取り組むか？ {#what-to-work-on}

[`status/ready-to-work-on`](https://github.com/blitz-js/blitz/labels/status%2Fready-to-work-on)ラベルの付いたイシューが最適なスタートポイントです。

適切な場合には、`good first issue`や`good second issue`というラベルも付けています。

興味を持ったイシューがあり、他に誰も取り組んでいなければ、そのイシューにコメントして「このイシューに取り組む」と伝えてください。ただし、数日以内に作業を開始できる場合のみにしてください。

質問があれば、イシュー内やDiscordで何でも聞いてください。お手伝いします！

Blitzjs.comのウェブサイトやドキュメントリポジトリにも[`ready to work on | help wanted`](https://github.com/blitz-js/blitzjs.com/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3A%22ready+to+work+on+%7C+help+wanted%22)ラベルの付いたイシューがあります。

### 常に歓迎されること {#things-that-are-always-welcome}

- テストの追加
- ドキュメントの改善
- エラーメッセージの改善
- ログの改善（より明確、より美しく）
- パフォーマンスやセキュリティの改善
- ブログ、ビデオ、コースなどの教育コンテンツ

他の方法で貢献したい場合は、Discordで相談してください！

どのような形であれ貢献した後は、[@all-contributors bot](https://allcontributors.org/docs/en/bot/usage)を使って自分をコントリビューターとして追加してください！

## 私たちのコードベースは庭のようなものです {#our-codebase-is-a-garden}

Blitzのコードベースはコミュニティガーデンのようなものです。美しい植物や野菜がたくさんありますが、すぐに雑草を見つけることができます！それを見つけたら、取り除いてください :) マイナーなリファクタリングは常に奨励されています。大きなリファクタリングをしたい場合は、まずイシューを立てるか、Discordで確認してください。おそらく同意するでしょう。

## PRを提出した後に期待できること {#what-to-expect-after-submitting-a-pr}

Blitzのメンテナーが通常数日以内にあなたのPRをレビューします。

ユーザーが目にするコードを変更した場合は、ドキュメントリポジトリにもPRを出してください。そうしないとPRのマージがブロックされます。

また、変更をカバーするためのテストを追加する必要があります。

すべての要件が満たされ、メンテナーがあなたのコードに満足したら、canaryブランチにマージされます。次のBlitzリリースに含まれます。

最後に、あなたは公式なBlitzコントリビューターとしてall-contributorsリストに追加されます。**おめでとうございます!!**

## プロジェクト管理 {#project-management}

すべてのイシューとPRを追跡するために、[GitHub Project Board](https://github.com/blitz-js/blitz/projects/4)を使用しています。

## コミット権限 {#commit-access}

Blitzリポジトリには、ある程度定期的に貢献している人には自由にコミット権限を付与します。これにより、フォークを使用せずに直接Blitzリポジトリにブランチをプッシュできます。

定期的に貢献していることに気づいたらアクセスを付与しますが、定期的にヘルプをしているのにまだアクセスが付与されていない場合は、アクセスを求めてもかまいません。

メインのBlitzリポジトリでは、PRのマージにはコードオーナーのレビューが必要です。

しかし、ドキュメントリポジトリでは、他の誰かがPRを承認すれば誰でもPRをマージできます。

## 開発環境のセットアップ {#development-setup}

Node 14を使用していることを確認してください — Nextは`node-sass`を使用しており、新しいNodeバージョンでは動作しません。

**1.** [Blitzリポ](https://github.com/blitz-js/blitz)をフォークします

**2.** フォークしたリポジトリをクローンします

```sh
# 以下のUSERNAMEをGitHubユーザー名に置き換えます
git clone git@github.com:USERNAME/blitz.git
cd blitz
```

**3.** （オプション、macOSのみ）`sodium-native`用の必須パッケージをインストールします

`sodium-native`に関連する問題が発生した場合は、以下のコマンドを実行して修正します：

```sh
brew install autoconf automake
```

**4.** 依存関係をインストールします

```sh
pnpm i
```

**5.** パッケージサーバーを起動します。これはパッケージ開発や例の開発のために**動作している必要があります**

```sh
pnpm dev
```

## テストの実行 {#running-tests}

### Blitz.jsのテスト {#blitz-tests}

#### ユニットテスト

`packages/`内のほとんどのBlitzパッケージにはvitestのユニットテストがあります。

- 単一のパッケージでテストを実行するには、そのパッケージフォルダー内で`pnpm test`を実行します（例：`packages/blitz-next`）
- リポジトリルートからすべてのユニットテストを実行するには`pnpm test`を実行します。（事前に`pnpm build`または`pnpm dev`を実行していることを確認してください）。注：これにより、統合テストも含めてすべてのテストが実行されます。

#### 統合テスト

Blitzの統合テストはルートの`integration-tests/`フォルダー内にあります。

Chromeバージョンに対応した`chromedriver`をインストールしていることを確認してください。以下のコマンドでインストールできます

- Mac OS Xでは`brew install --cask chromedriver`
- Windowsでは`chocolatey install chromedriver`
- または、対応するChromeバージョンの`chromedriver`バイナリを[chromedriver repo](https://chromedriver.storage.googleapis.com/index.html)から手動でダウンロードして、`<blitz-repo>/node_modules/.bin`に追加します

リポジトリルートからすべてのテスト（ユニットテストと統合テストの両方）を実行するには、`pnpm test`を実行します。

統合テストフォルダー内で`blitz dev`を実行して統合アプリを手動で実行できます（例：`integration-tests/auth/`）。

### アプリ内でBlitzの開発バージョンをテストする {#testing-development-version-of-blitz}

<Card type="info">

現在、Blitzのローカル開発バージョンをテストするには、`blitz/apps/`フォルダー内のアプリをテストできます。ここでは、Blitzの依存関係は自動的にローカル開発バージョンを使用します。開発テストに使用します。同時に`blitz`フォルダー内で`pnpm dev`を実行していることを確認してください。

</Card>

#### Blitz CLIをリンクする（オプション）

以下のコマンドで開発CLIをローカルバイナリとしてリンクし、どこでもテストに使用できるようにします。

```sh
pnpm link ./packages/blitz
```

## リリースの実施 {#doing-a-release}

[changesets](https://github.com/changesets/changesets)パッケージと、以下にあるGitHubアクションのフォークを使用しています：[GitHub - blitz-js/changesets-action](https://github.com/blitz-js/changesets-action)。ユーザーが目にするPRにはすべてchangesetが必要です。そのPRが承認され`main`ブランチにマージされると、「Version packages (beta)」という名前の"release" PRが自動的に作成または更新されます。このPRは、各内部パッケージのチェンジログと各Blitzパッケージのバージョンを`package.json`ファイル内で増分します。

自動生成されたPRが十分なchangesetを蓄積したら、このPRを`main`ブランチにマージしてリリースを実行します。リリースアクションをトリガーするには、このPRをマージ保護ルールを無視してスカッシュマージする必要があります。これは、githubアクションによって生成されたPRではCIが実行されないためです。このPRをマージすると、リリースアクションが生成され、npmに公開され、GitHubにリリースが作成されます。

GitHubの自動生成されたリリースには、いくつかの手動クリーンアップが必要です。一部の"フラッフ"を削除することをお勧めします：

```
Updated dependencies [3b213a3]
	@blitzjs/rpc@2.0.0-beta.18
```

また、安定版リリースをまだ行っていないため、`changesets`パッケージでプレリリースモードに設定されていることに注意してください。したがって、GitHubのリリースを編集する際には「Set as a pre-release」チェックボックスを解除し、「Set as the latest release」チェックボックスを選択する必要があります。

`changesets`パッケージでのプレリリースモードにあるため、npmに公開する際には各パッケージが`latest`タグでタグ付けされるようにする必要があります。これには適切なnpm権限が必要です。_これは主に内部チーム向けの作業です。_ 各パッケージに対して以下を実行します：

```shell
npm dist-tag add blitz@2.0.0-beta.18 latest
npm dist-tag add @blitzs/auth@2.0.0-beta.18 latest
npm dist-tag add @blitzs/rpc@2.0.0-beta.18 latest
npm dist-tag add @blitzs/next@2.0.0-beta.18 latest
```

`changesets`プレリリースモードに関する詳細情報は、こちらのドキュメントをご覧ください：
[changesets/prereleases.md at main · changesets/changesets · GitHub](https://github.com/changesets/changesets/blob/main/docs/prereleases.md)

## トラブルシューティング {#troubleshooting}

ここに記載されるべき問題に遭遇した場合は、PRを提出してください！
❤️

#### Gitエラー

コミットできない場合や`command not found`や`stdin is not a tty`のようなエラーが発生する場合、それはおそらくHuskyのエラーです！彼らの[トラブルシュートガイド](https://typicode.github.io/husky/#/?id=troubleshoot)を確認してください。

#### Preconstructエラー

WindowsでPreconstructを実行しようとした際にシンボリックリンクやEPERMエラーに遭遇する場合は、[Windows Developer Mode](https://www.howtogeek.com/292914/what-is-developer-mode-in-windows-10/)を有効にする必要があるかもしれません。これにより、Preconstructがシンボリックリンクを作成できるようになります。

#### Windowsでファイルが見つからない

`pnpm build`を実行してもファイルが見つからないエラーが発生する場合は、`core.symlinks`設定を`true`にしてリポジトリを再度クローンしてみてください： `git clone -c core.symlinks=true <URL>`.
