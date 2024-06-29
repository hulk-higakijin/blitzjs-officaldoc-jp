    ```
    ---
    title: 自分だけのレシピを書く
    sidebar_label: レシピの作成
    ---

    Blitz Recipesの素晴らしい力は、Blitzリポジトリの公式レシピに限られたものではありません。レシピを構築するためのAPIは、`blitz/installer`から[`blitz`](https://npm.im/blitz)パッケージを介して公開されており（変更の可能性がありますが）、これをインストールしてカスタムレシピを書けるようになっています。既存のファイルを変換するための[`jscodeshift`](https://github.com/facebook/jscodeshift)の力と組み合わせれば、完全に自動化されたコード移行がBlitzエコシステムの一級市民となります。

    ### セットアップ {#setup}

    独自のレシピを作成するには、新しいパッケージを作成し、いくつかの依存関係をインストールします。既存のファイルを修正するための**変換**ステップを使用する場合は、`jscodeshift`の依存関係が必要になります。新しいファイルを作成するか依存関係を追加する場合には、`blitz`だけが必要です。

    レシピのテストを書く場合には、ビルドとテストのセットアップが必要です。ビルドとテストの実行には[`tsdx`](https://github.com/jaredpalmer/tsdx)とVitestをお勧めします。

    ```bash
    pnpm add blitz jscodeshift @types/jscodeshift
    ```

    レシピAPIはすべて`RecipeBuilder`ファクトリを中心に展開されます。Blitzは`package.json`の`main`フィールドに参照されているファイルがデフォルトでレシピをエクスポートするものと仮定しているため、それをセットアップしましょう。

    ```json
    // package.json
    {
      "main": "index.ts"
    }
    ```

    ```typescript
    // index.ts
    import { RecipeBuilder } from "blitz/installer"

    export default RecipeBuilder().build()
    ```

    ### メタデータの追加 {#adding-metadata}

    レシピの実際のステップに加えて、開発者に対してレシピに関するメタデータを提供することを要求します。これにより、ユーザーが何をインストールしているのか、およびサポートが必要な場合にどこを見ればよいのかを表示することができます。

    ```typescript
    RecipeBuilder()
      .setName("My Package")
      .setDescription(
        "インストールされるものの詳細情報."
      )
      .setOwner("架空の著者 <blitz@blitzjs.com>")
      .setRepoLink("https://github.com/fake-author/my-recipe")
      .build()
    ```

    これは非常に良いスタートであり、実際、ユーザーが`blitz install`を介して実行できる実行可能なレシピを作成するために必要なすべてです。しかし、インストーラーのフレームワークが実行するステップが実際にないため、あまり役に立ちません。引き続き、実行可能なさまざまなアクションについて学んでください。

    ### 共通API {#common-api}

    各アクションタイプには、レシピの「ステップ」を定義するための共通インターフェースがあります。これにより、ユーザーエクスペリエンスの一貫性が確保され、快適なインストール体験を提供できるようになります。追加する各ステップには、インストールの進行を内部的に追跡するために使用される文字列のID、表示名、およびステップが何をしているのかの説明が必要です。

    ```typescript
    interface Config {
      stepId: string
      stepName: string
      explanation: string
    }
    ```

    最終的に、レシピのライフサイクルにフックを提供し、一部のメタデータ（`stepId`など）を重要視することを期待しています。

    ### 依存関係の追加 {#adding-dependencies}

    最初にできるアクションは、ユーザーのアプリケーションに依存関係を追加することです。このステップタイプは、ユーザーが`yarn`や`npm`を使用しているかどうかを自動的に検出し、適切なツールを使用します。設定は非常に簡単で、パッケージのリスト、バージョン、およびそれが`devDependency`としてインストールされるかどうかを受け入れます。

    ```typescript
    builder.addAddDependenciesStep({
      stepId: "addDeps",
      stepName: "npm 依存関係の追加",
      explanation: `Tailwind ライブラリ自体、そして生産バンドルから未使用のスタイルを削除するための PostCSS をインストールします。`,
      packages: [
        { name: "tailwindcss", version: "1" },
        { name: "postcss-preset-env", version: "latest", isDevDep: true },
      ],
    })
    ```

    ### ファイルの追加 {#adding-files}

    レシピの非常に強力な部分の1つは、テンプレートからファイルを生成できることです。

    読み書き両方に自然なカスタムテンプレート言語を使用しています。[テンプレートの書き方についてはテンプレートドキュメントをご覧ください](templates)。

    `templateValues`設定を提供することで、テンプレートに埋め込む値のハードコードされたオブジェクトやオブジェクトを返す関数を提供できます。関数には、`blitz install myrecipe --someConfig=false`のように`blitz install`に渡された追加の引数が渡されます。テンプレートファイルはレシピのファイル構造内の任意の場所に配置でき、レシピ定義の一部としてパスを提供します。

    ```typescript
    import { join } from "path"

    builder.addNewFilesStep({
      stepId: "addStyles",
      stepName: "基本的な Tailwind CSS スタイルの追加",
      explanation: `次に、実際にスタイルシートを作成する必要があります！これらのスタイルシートは、アプリ全体のグローバルスタイルを含むように変更するか、コンポーネント内でクラス名を使用することで済ませることができます。`,
      targetDirectory: "./src",
      templatePath: join(__dirname, "templates", "styles"),
      templateValues: {},
    })
    ```

    ### 既存のファイルの修正 {#modifying-existing-files}

    Blitzレシピの最も強力な部分と言える部分で、JSCodeShiftを使用して既存のファイルを修正する変換を書けます。変換関数には、選択したファイルを表すAST、ノードの新しいノードを作成するためのオブジェクト、およびノードの型チェックやアサーションを支援するオブジェクトが渡されます。[ASTExplorer](https://astexplorer.net)は、AST構造に慣れ親しみ、変換を試すのに適した場所です。最良の結果を得るには、パーサ設定で`@babel/parser`を、変換設定で`jscodeshift`を使用してください。

    Blitzは一般的なケースのためのいくつかの事前定義された変換を提供しますが、任意のJavaScriptファイルを修正するためのカスタム変換を書くこともできます。また、`_app.tsx`や`next.config.js`のような一般的なファイルにアクセスするための便利なユーティリティを提供しています。ファイルパスがグロブパターンである場合、インストーラープロセスはパターンに一致するファイルを選択するためにユーザーにプロンプトを表示します。

    ```typescript
    import { addImport, paths } from "blitz/installer"
    import j from "jscodeshift"

    builder.addTransformFilesStep({
      stepId: "importStyles",
      stepName: "スタイルシートのインポート",
      explanation: `最終的に、作成したスタイルシートをアプリケーションに読み込みます。今回は document.tsx に配置しますが、tailwindでアプリの一部だけをスタイルする場合は、コンポーネントツリーの下部でスタイルをインポートすることができます。`,
      singleFileSearch: paths.app(),
      transform(program: j.Collection<j.Program>) {
        const stylesImport = j.importDeclaration(
          [],
          j.literal("src/styles/index.css")
        )
        return addImport(program, stylesImport)!
      },
    })
    ```

    変換はAST上で実行される自己完結型の関数であるため、インストーラーのこの部分をユニットテストすることができます。これは信頼性のために非常に役立ちます。`jscodeshift`を直接使用し、スナップショットテストと組み合わせることで、テストは迅速に書けます：

    ```typescript
    import j from "jscodeshift"
    import { addImport, customTsParser } from "blitz/installer"

    const sampleFile = `export default function Comp() {
      return <div>hello!</div>;
    })`
    expect(
      addImport(
        j(sampleFile, {
          parser: customTsParser,
        }),
        newImport
      ).toSource()
    ).toMatchSnapshot()
    ```

    #### 非JSファイルの修正

    すべての修正が`jscodeshift`を使用して行えるわけではありません。他の変換が必要な場合は、`transformPlain`を使用することができます：

    ```ts
    builder.addTransformFilesStep({
      // ...
      singleFileSearch: "README.md",
      transformPlain(readme: string) {
        return readme + "\n" + "Paul Plain was here!"
      },
    })
    ```

    このステップは、ユーザーのREADME.mdに"Paul Plain was here!"を追加します。

    #### カスタムメッセージの作成

    簡単なメッセージを表示したり、いくつかの指示を与えたいときには、`printMessage`でそれを達成できます。

    ```ts
    builder.printMessage({
      successIcon: "ℹ️",
      stepId: "oops",
      stepName: "協力者が必要です！",
      message: "このレシピが気に入ったら、手を貸してください！",
    })
    ```

    このステップは、"このレシピが気に入ったら、手を貸してください！"というメッセージを端末に表示し、ユーザーが次のステップに進むのを待ちます。

    #### カスタムステップアイコン

    レシピをさらにカスタマイズしたい場合は、ステップにオプションの`successIcon`パラメーターを渡すことができます。

    ```ts
    builder.addTransformFilesStep({
      successIcon: "🔧",
      // ...
    })
    ```

    #### Prismaスキーマの修正

    多くのレシピは、レシピの効果を適用するためにschema.prismaファイルでモデルを追加または変更する必要があります。`transformPlain`で文字列変換を使用してスキーマを操作することもできますが、多くのレシピは最初にスキーマに関する何かをクエリする必要があります。たとえば、[リレーション](https://www.prisma.io/docs/concepts/components/prisma-schema/relations/)の対象となるモデルを特定したり、フィールドが既に存在するかどうかを検出する必要があります。

    便利なことに、一般的なスキーマの変更のためにいくつかのユーティリティが事前に書かれています：

    ##### Enumを作成する

    ```ts
    singleFileSearch: paths.prismaSchema(),
    transformPlain(source: string) {
      // Enum Role を USER と ADMIN の2つの値で作成
      return addPrismaEnum(source, {
        type: "enum",
        name: "Role",
        enumerators: [
          {type: "enumerator", name: "USER"},
          {type: "enumerator", name: "ADMIN"},
        ],
      })
    }
    ```

    ##### モデルへのフィールドの追加

    ```ts
    singleFileSearch: paths.prismaSchema(),
    transformPlain(source: string) {
      // "Project"モデルにフィールド "name String @unique" を追加
      return addPrismaField(source, "Project", {
        type: "field",
        name: "name",
        fieldType: "String",
        optional: false,
        attributes: [{type: "attribute", kind: "field", name: "unique"}],
      })
    }
    ```

    ##### ジェネレータを作成する

    ```ts
    singleFileSearch: paths.prismaSchema(),
    transformPlain(source: string) {
      // nexus-prisma 用の prisma ジェネレータを作成
      return addPrismaGenerator(source, {
        type: "generator",
        name: "nexusPrisma",
        assignments: [
          {type: "assignment", key: "provider", value: '"nexus-prisma"'},
        ],
      })
    }
    ```

    ##### モデルを作成する

    ```ts
    singleFileSearch: paths.prismaSchema(),
    transformPlain(source: string) {
      // Project という名前の prisma モデルを作成
      return addPrismaModel(source, {
        type: "model",
        name: "Project",
        properties: [{type: "field", name: "name", fieldType: "String"}],
      })
    }
    ```

    ##### モデルにインデックスなどの属性を追加する

    ```ts
    singleFileSearch: paths.prismaSchema(),
    transformPlain(source: string) {
      // Project モデルにインデックス属性 "@@index([name])" を作成
      return addPrismaModelAttribute(source, "Project", {
        type: "attribute",
        kind: "model",
        name: "index",
        args: [
          {type: "attributeArgument", value: {type: "array", args: ["name"]}},
        ],
      })
    }
    ```

    ##### スキーマのデータソースを設定する

    prisma スキーマは1つのデータソースしか持てないため、"add" ユーティリティは存在せず、これはスキーマの現在のデータソースを置き換えます。

    ```ts
    singleFileSearch: paths.prismaSchema(),
    transformPlain(source: string) {
      // データソースを postgresql に設定
      return setPrismaDataSource(source, {
        type: "datasource",
        name: "db",
        assignments: [
          {type: "assignment", key: "provider", value: '"postgresql"'},
          {
            type: "assignment",
            key: "url",
            value: {type: "function", name: "env", params: ['"DATABASE_URL"']},
          },
        ],
      })
    }
    ```

    ##### produceSchema を使用したカスタムスキーマ変換

    提供されたヘルパーがレシピには十分に柔軟でない場合、`produceSchema`ユーティリティ関数を使用して prisma スキーマファイルをパースし、カスタム変換を適用できます。これを使用すると、スキーマを JSON オブジェクト形式に変換し、JavaScript を使用して変更を加え、変更を適用したスキーマを文字列として戻します。上記のすべてのヘルパーは、`produceSchema`を使用して実装されています。

    ```ts
    builder.addTransformFilesStep({
      // ...
      singleFileSearch: paths.prismaSchema(),
      transformPlain(source: string) {
        return produceSchema(source, (schema) => {
          // "User" という名前のモデルを見つける
          const model = schema.list.find(function (item): item is Model {
            return item.type === "model" && item.name === "User"
          }) as Model
          if (!model) return

          // "User" モデルの "email" という名前のフィールドを見つける
          const field = model.properties.find(function (
            property
          ): property is Field {
            return property.type === "attribute" && property.name === "email"
          })
          if (!field) return

          // "email" に "@unique" 属性を追加
          field.attributes?.push({
            type: "attribute",
            kind: "field",
            name: "unique",
          })
        })
      },
    })
    ```

    スキーマ変換のベストプラクティスは、冪等性を保つことです。つまり、スキーマに対して既に変更が行われている場合は、変更を行おうとしないことです。たとえば、モデルに既に存在するフィールドを追加しないようにします。

    スキーマファイルがどのようにパースされるかを確認するには、
    [こちらをクリック](https://github.com/MrLeebo/prisma-ast)してください。

    ### Next.js の設定の修正 {#modifying-next-config}

    一部のレシピは、既存のNext.js