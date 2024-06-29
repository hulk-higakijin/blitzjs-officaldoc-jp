---
title: 貢献する方法
---

👋 Blitzに貢献することに興味を持っていただき、とても嬉しいです！オープンソースの経験がなくても、始めるお手伝いをします :)

## 最初にすること {#first-things-first}

1. オープンソースは初めてですか？[GitHubでオープンソースプロジェクトに貢献する方法](https://egghead.io/courses/how-to-contribute-to-an-open-source-project-on-github)をご覧ください。
2. [Blitz行動規範](./code-of-conduct)を熟読してください。
3. [コミュニティの運営方法](./how-the-community-operates)を学びましょう。

## 何に取り組むべき？ {#what-to-work-on}

[`status/ready-to-work-on`](https://github.com/blitz-js/blitz/labels/status%2Fready-to-work-on)ラベルが付いている問題が始めるのに最適です。

適宜、`good first issue`や`good second issue`とラベル付けされた問題もあります。

興味がある問題を見つけたら、誰も取り組んでいない場合、その問題にコメントして取り組むことを宣言してください。ただし、数日以内に作業を開始できる場合に限ります。

必要に応じて、問題内やDiscordで質問してください。喜んでお手伝いします！

Blitzjs.comのウェブサイトおよびドキュメントリポジトリにも[`ready to work on | help wanted`](https://github.com/blitz-js/blitzjs.com/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc+label%3A%22ready+to+work+on+%7C+help+wanted%22)ラベルが付いた問題があります。

### いつでも歓迎するもの {#things-that-are-always-welcome}

- テストの追加
- ドキュメントの改善
- エラーメッセージの改善
- ロギングの改善（より明確、より美しい）
- パフォーマンスやセキュリティの改善
- ブログ、ビデオ、コースなどの教育コンテンツ

他の方法で貢献したい場合は、Discordでお問い合わせください！

どのような形で貢献しても、[@all-contributors bot](https://allcontributors.org/docs/en/bot/usage)を使って自分を貢献者として追加してください！

## コードベースは庭園のようなもの {#our-codebase-is-a-garden}

Blitzのコードベースはコミュニティガーデンのようなものです。美しい植物や野菜がたくさんありますが、すぐに雑草を見つけることができます！雑草を見つけたら、取り除いてください :) 小規模なリファクタリングは常に奨励されます。大規模なリファクタリングを行いたい場合は、まず問題を開くか、Discordで確認するのがベストです。ほとんどの場合、同意します。

## PRを提出した後に期待すること {#what-to-expect-after-submitting-a-pr}

Blitzのメンテナーが通常数日以内にPRをレビューします。

ユーザーに影響を与えるコードを変更した場合は、ドキュメントリポジトリにもPRを提出してください。これを行わないと、PRがマージされるのをブロックします。

変更した内容をカバーするテストも追加する必要があります。

すべての要件が満たされ、メンテナーがコードに満足したら、それをcanaryブランチにマージします。次のBlitzリリースに含まれます。

最後に、Blitzの公式貢献者としてall-contributorsリストに追加されます。**おめでとうございます！！**

## プロジェクト管理 {#project-management}

すべての問題とPRを追跡するために、[GitHub Project Board](https://github.com/blitz-js/blitz/projects/4)を使用しています。

## コミットアクセス {#commit-access}

Blitzリポジトリへの定期的な貢献者には、リベラルなコミットアクセスを提供します。これにより、フォークを使用せずにBlitzリポジトリに直接ブランチをプッシュできます。

定期的に貢献していることに気付いた場合、アクセスを提供することがあります。しかし、定期的に助けているがまだアクセスを得ていない場合は、アクセスを要求することもできます。

メインのBlitzリポジトリでは、コードオーナーによるコードレビューがPRをマージするために必要です。

しかし、ドキュメントリポジトリでは、他の人がPRを承認した後、誰でもPRをマージできます。

## 開発セットアップ {#development-setup}

Node 14を使用していることを確認してください — Nextは新しいNodeバージョンでは動作しない`node-sass`を使用します。

**1.** [blitzリポジトリ](https://github.com/blitz-js/blitz)をフォークします。

**2.** フォークしたリポジトリをクローンします。

```sh
# 以下のUSERNAMEをGitHubユーザー名に置き換えてください。
git clone git@github.com:USERNAME/blitz.git
cd blitz
```

**3.** （オプション、macOSのみ）`sodium-native`用の必要なパッケージをインストールします。

`sodium-native`に関連する問題が発生した場合は、次のコマンドを実行して修正してください：

```sh
brew install autoconf automake
```

**4.** 依存関係をインストールします。

```sh
pnpm i
```

**5.** パッケージサーバーを起動します。これはパッケージ開発や例の開発のために**必ず実行する必要があります**。

```sh
pnpm dev
```

## テストの実行 {#running-tests}

### Blitz.jsテスト {#blitz-tests}

#### 単体テスト

`packages/`内のほとんどのBlitzパッケージにはvitest単体テストがあります。

- 単一パッケージのテストを実行するには、そのパッケージフォルダー（例えば`packages/blitz-next`）内で`pnpm test`を実行します。
- リポジトリのルートからすべての単体テストを実行するには、`pnpm test`を実行します。（これを実行する前に`pnpm build`または`pnpm dev`を実行していることを確認してください）。注：これにより、統合テストを含むすべてのテストが実行されます。

#### 統合テスト

Blitzの統合テストはルートの`integration-tests/`フォルダー内にあります。

Chromeバージョン用の`chromedriver`がインストールされていることを確認してください。以下の方法でインストールできます：

- Mac OS Xでは`brew install --cask chromedriver`
- Windowsでは`chocolatey install chromedriver`
- または、[chromedriverリポジトリ](https://chromedriver.storage.googleapis.com/index.html)からインストールされているChromeバージョンに一致するバージョンを手動でダウンロードし、バイナリを`<blitz-repo>/node_modules/.bin`に追加します（一致するバージョンがない場合、下位のバージョンをダウンロードしますが、上位のバージョンは避けてください）。

リポジトリのルートから`pnpm test`を実行して、すべてのテスト（単体テストと統合テストの両方）を実行できます。

統合テストフォルダー内で`blitz dev`を実行して統合アプリを手動で実行できます。例えば、`integration-tests/auth/`のように。

### Blitzの開発バージョンをアプリ内でテストする {#testing-development-version-of-blitz}

<Card type="info">

現在、Blitzのローカル開発バージョンをテストするには、`blitz/apps/`フォルダー内のアプリをテストできます。ここでは、Blitzの依存関係が自動的にローカル開発バージョンを使用します。これは開発テスト用に使用します。また、同時に`blitz`フォルダー内で`pnpm dev`を実行していることを確認する必要があります。

</Card>

#### Blitz CLIをリンクする（オプション）

以下を

実行して、開発CLIをローカルバイナリとしてリンクし、どこでもテストで使用できるようにします。

```sh
pnpm link ./packages/blitz
```

## リリースの実施 {#doing-a-release}

[changesets](https://github.com/changesets/changesets)パッケージとそのGitHubアクションのフォークバージョンを使用しています。こちらで見つけることができます：[GitHub - blitz-js/changesets-action](https://github.com/blitz-js/changesets-action)。ユーザーに影響を与えるすべてのPRにはchangesetが必要です。そのPRが承認され、`main`ブランチにマージされると、`Version packages (beta)`という名前の「リリース」PRが自動的に作成または更新されます。このPRは各内部パッケージのchangelogと各blitzパッケージのバージョンを`package.json`ファイル内でインクリメントして更新します。

自動生成されたPRが十分なchangesetを蓄積すると、このPRを`main`ブランチにマージすることでリリースを実行します。リリースアクションをトリガーするには、要件を満たすのを待たずにPRをsquash and mergeする必要があります。これは、CIがGitHubアクションによって生成されたPRでは実行されないためです。このPRをマージすると、リリースアクションが生成され、npmに公開され、GitHubにリリースが作成されます。

自動生成されたGitHubのリリースには、いくつかの手動のクリーンアップが必要です。次のような「無駄」を削除することをお勧めします：

```
Updated dependencies [3b213a3]
	@blitzjs/rpc@2.0.0-beta.18
```

まだ安定リリースを行っていないため、`changesets`パッケージでプレリリースモードに設定されていることにも注意してください。したがって、GitHubでリリースを編集する際に「Set as a pre-release」チェックボックスをオフにし、「Set as the latest release」チェックボックスを選択する必要があります。

`changesets`パッケージのプレリリースモードのカテゴリにある場合、npmに公開されたときに各パッケージが`latest`でタグ付けされるようにする必要があります。これは主に内部チーム向けです。各パッケージに対して次を実行します：

```shell
npm dist-tag add blitz@2.0.0-beta.18 latest
npm dist-tag add @blitzs/auth@2.0.0-beta.18 latest
npm dist-tag add @blitzs/rpc@2.0.0-beta.18 latest
npm dist-tag add @blitzs/next@2.0.0-beta.18 latest
```

`changesets`のプレリリースモードに関する詳細情報は、こちらのドキュメントを参照してください：[changesets/prereleases.md at main · changesets/changesets · GitHub](https://github.com/changesets/changesets/blob/main/docs/prereleases.md)

## トラブルシューティング {#troubleshooting}

ここに記載すべき問題に直面した場合は、PRを提出してください！❤️

#### Gitエラー

コミットできませんか？`command not found`や`stdin is not a tty`のようなエラーが発生しますか？それはおそらくHuskyのエラーです！[トラブルシュートガイド](https://typicode.github.io/husky/#/?id=troubleshoot)を確認してみてください。

#### Preconstructエラー

WindowsでPreconstructを実行しようとすると、シンボリックリンクやEPERMエラーが発生する場合、Preconstructがシンボリックリンクを作成できるように[Windows開発者モード](https://www.howtogeek.com/292914/what-is-developer-mode-in-windows-10/)を有効にする必要があるかもしれません。

#### Windowsでのファイル不足

`pnpm build`を実行した後でもファイルが不足しているというエラーが発生する場合は、次のように`core.symlinks`を`true`に設定してリポジトリを再クローンしてみてください：`git clone -c core.symlinks=true <URL>`。
