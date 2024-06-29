---
title: ローカルでPostgresを実行する
---

ローカルでPostgresを実行するには、基本的に2つの方法があります。

1. コンピュータにネイティブにインストールする
2. Dockerを使用する

ほとんどの人はDockerを使用せずにネイティブにPostgresを実行しています。これは、簡単であり、バックグラウンドで常に実行されるためです。また、Dockerは追加のリソースを使用します。

## コンピュータにネイティブにインストールする {#natively-on-your-computer}

### Macの場合 {#on-mac}

非常に人気のある方法が2つあります。

#### Homebrewでインストール

1. [Homebrew](https://brew.sh/) がインストールされていることを確認してください。
2. `brew install postgresql` を実行します。
3. `brew services start postgresql` を実行します。

これで完了です！

#### Postgres.appでインストール

1. [Postgres.app](https://postgresapp.com/) にアクセスします。
2. アプリをダウンロードしてインストールします。
3. アプリを開いて「初期化」をクリックし、新しいサーバーを作成します。

これで準備完了です！

### Windowsの場合 {#on-windows}

最も簡単な方法は、[Postgresのウェブサイト](https://www.postgresql.org/download/windows/) からインストーラをダウンロードして実行することです。

## Dockerを使用する {#via-docker}

1. プロジェクトのルートに以下の内容で`docker-compose.yml`ファイルを作成します。

```yaml
version: "3.7"

services:
  db:
    image: postgres:latest
    volumes:
      - data:/var/lib/postgresql/data
    env_file: ./.env.local # ここでは既存の.env.localファイルを使用しています
    ports:
      - "5432:5432"

  db-test:
    image: postgres:latest
    env_file: ./.env.test.local
    ports:
      - 5433:5432

volumes:
  data:
```

2. `.env.local` ファイルに、Dockerが必要とする3つの新しい環境変数を追加します。

```bash
POSTGRES_USER=your_user
POSTGRES_PASSWORD=your_password
POSTGRES_DB=your_database_name
```

これらの値を使用して、`.env.local` 内の `DATABASE_URL` を例えば次のように更新します。
`postgresql://your_user:your_password@localhost:5432/your_database_name`
また、`.env.test.local` では次のように設定します。
`postgresql://your_user:your_password@localhost:5433/your_database_name`

テストデータベースはポート `5433` を使用している点に注意してください。

3. `package.json` を変更して、Blitzの前にデータベースを起動します。

```json
"scripts": {
    "predev": "docker-compose up -d",
    "dev": "blitz dev",
}
```

4. 新しいデータベースを起動し、`docker-compose up -d` と
   `blitz prisma migrate dev` を実行して最新のマイグレーションにします。
