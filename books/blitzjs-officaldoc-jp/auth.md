    ===
    ---
    title: 認証の概要
    sidebar_label: 概要
    ---

    新しいBlitzアプリは、ユーザーのサインアップ、ログイン、ログアウトを含む認証と認可がデフォルトでセットアップされています。`db/schema.prisma` ファイルには `User` と `Session` モデルがあり、認証コードは `src/auth/` フォルダーにあります。

    Blitzには、メール/パスワード認証および任意のサードパーティプロバイダーと連携する組み込みのセッション管理機能があります。

    Blitzのセッション管理は、最先端の [SuperTokensライブラリ](https://supertokens.com) と同じアプローチに従っています。SuperTokensのCTOである[Rishabh Poddar](https://twitter.com/rishpoddar) が我々の実装の設計と監督を行いました。Rishabhの助けに非常に感謝しています！🙏

    ===