    ---
    title: blitz start
    sidebar_label: blitz start
    ---

    **エイリアス: `blitz s`**

    Blitzのプロダクションサーバーを開始します。

    これは、環境変数のロードを改善するためにNext CLI上に設けられた非常に薄いラッパーです。

    ```bash
    blitz dev -e staging
    ```

    以下を行います:

    1. 指定した`-e X`フラグに基づいて適切な`.env.X`および`.env.X.local`ファイルをロードします。例: `-e staging`は` .env.staging`をロードします。
    2. `process.env.APP_ENV`を環境名に設定します。例: `-e staging` => `APP_ENV=staging`。
    3. それ以外のすべての引数をnextビルドコマンドに直接渡します。

    #### オプション

    | オプション      | ショートハンド | 説明                                                                                       | デフォルト     |
    | ------------ | --------- | ------------------------------------------------------------------------------------- | ------------- |
    | `--hostname` | `-H`      | サーバーに使用するホスト名を設定します。                                                        | `"localhost"` |
    | `--port`     | `-p`      | サーバーがリスンするポートを設定します。                                                       | `3000`        |
    | `--inspect`  |           | Node.jsインスペクターを有効にします。                                                          | `false`       |
    | `--env`      | `-e`      | アプリ環境名を設定します。[詳細はこちらを読んでください](/docs/custom-environments#custom-environments). | なし          |

    #### 例

    <Card type="note">

    `blitz start`を実行する前に`blitz build`を実行することを確認してください

    </Card>

    ```bash
    blitz start
    blitz start -H 127.0.0.1 -p 5632
    ```