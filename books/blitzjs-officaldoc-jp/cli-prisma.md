---
title: blitz prisma
sidebar_label: blitz prisma
---

**エイリアス: `blitz p`**

これはすべての環境変数をロードし、その後すべての引数を[Prisma CLI](https://www.prisma.io/docs/reference/api-reference/command-reference)にプロキシします。

望むことにプリズマが完全な環境サポートを追加してくれることで、最終的にはこれを削除できるようになることでしょう :)

#### オプション

| オプション | ショートハンド | 説明                                                                                          | デフォルト |
| ---------- | -------------- | --------------------------------------------------------------------------------------------- | ---------- |
| `--env`     | `-e`           | アプリケーションの環境名を設定します。[詳細はこちら](/docs/custom-environments#custom-environments). | なし       |

#### 例

```
blitz prisma generate
blitz prisma migrate dev
blitz prisma migrate deploy
```
