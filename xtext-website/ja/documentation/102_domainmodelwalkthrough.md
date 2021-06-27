---
layout: documentation
title: 15分チュートリアル
part: はじめよう
---

# {{page.title}} {#domain-model-walkthrough}

このチュートリアルでは、Rails、GrailsやSpring Rooのように、エンティティとプロパティをモデル化するための小さなドメイン固有言語を実装します。この構文はとても示唆に富んでいます。

```domainexample
datatype String

entity Blog {
    title: String
    many posts: Post
}

entity HasAuthor {
    author: String
}

entity Post extends HasAuthor {
    title: String
    content: String
    many comments: Comment
}

entity Comment extends HasAuthor {
    content: String
}
```

あなたのマシーンにXtextをインストールした後、Eclipseを起動し、新規workspaceをセットアップします。

## 新規Xtextプロジェクトの作成

まず初めに、Eclipseプロジェクトを作る必要があります。Eclipseウィザードから以下の操作を行います。

*File &rarr; New &rarr; Project... &rarr; Xtext &rarr; Xtext project*

意味のあるプロジェクト名、言語名、そしてファイル拡張子を選びます。例えば

|:---|:---|
|**プロジェクト名:**|org.example.domainmodel|
|**言語名:**|org.example.domainmodel.Domainmodel|
|**DSL-ファイル拡張子:**|dmodel|

プロジェクトを作成するため *Finish* を押してください。

![](../../documentation/images/30min_wizard.png)

ウィザードを正常に終了した後、ワークスペースには5つの新しいプロジェクトが作成されます。

|:---|:---|
|org.example.domainmodel| 文法定義と全ての言語固有コンポーネント(パーサー、字句解析器、リンカー、バリデーションなど)|
|org.example.domainmodel.ide| プラットフォームに依存しないIDE機能 (例 コンテントアシスタントサービス)|
|org.example.domainmodel.tests| 言語のユニットテスト|
|org.example.domainmodel.ui| Eclipseエディターと、ワークベンチ関連機能|
|org.example.domainmodel.ui.tests| Eclipseエディター用UIテスト|

![](../../documentation/images/30min_initialprojectlayout.png)

## 文法を書く

エディターで 文法ファイル*Domainmodel.xtext* を開くと、ウィザードが自動的に開きます。文法ファイルにはシンプルな *Hello World* 文法が含まれています。

```xtext
grammar org.example.domainmodel.Domainmodel with
                                      org.eclipse.xtext.common.Terminals

generate domainmodel "http://www.example.org/domainmodel/Domainmodel"

Model:
    greetings+=Greeting*;
  
Greeting:
    'Hello' name=ID '!';
```

さっそく、文法定義をエンティティ言語で置き換えてみましょう。

```xtext
grammar org.example.domainmodel.Domainmodel with
                                      org.eclipse.xtext.common.Terminals

generate domainmodel "http://www.example.org/domainmodel/Domainmodel"

Domainmodel:
    (elements+=Type)*;

Type:
    DataType | Entity;

DataType:
    'datatype' name=ID;

Entity:
    'entity' name=ID ('extends' superType=[Entity])? '{'
        (features+=Feature)*
    '}';

Feature:
    (many?='many')? name=ID ':' type=[Type];
```

文法ルールの意味を詳しく見ていきましょう。

1.  文法の最初のルールは常にスタートルールとして使用されます。
    
    ```xtext
    Domainmodel:
        (elements+=Type)*;
    ```

    これは、*Domainmodel* が任意の数の*Type*を持ち、それらを `elements`と呼ばれる特性に追加 (`+=`) することを表しています。
1.  *Type* ルールは*DataType* ルール もしくは (`|`) *Entity* ルールに委任します。

    ```xtext
    Type:
        DataType | Entity;
    ```

1.  *DataType* ルールは `'datatype'`キーワードから始まり、*ID* と呼ばれるルールによってパースされる識別子が続きます。*ID* ルールは上位文法 *org.eclipse.xtext.common.Terminals* で定義され、一つの単語(識別子)をパースします。 ルールの呼び出し箇所で *F3* を押すことで、宣言箇所にジャンプできます。*ID* の戻り値は 特性`name`に割り当てられます(`=`)。

    ```xtext
    DataType:
        'datatype' name=ID;
    ```

1.  *Entity* ルールもキーワードの定義から始まり、次に名前が続きます。

    ```xtext
    Entity :
        'entity' name=ID ('extends' superType=[Entity])? '{'
            (features+=Feature)*
        '}';
    ```

    次に かっこで囲まれたオプション (`?`) の`extends` 句があります。 特性 `superType` は クロスリファレンス ( 角かっこに注意 )なので、 パーサールール *Entity* はここでは呼び出されず、一つの識別子だけが (*ID* ルール) がパースされます。`superType` で参照する実際の *Entity* 割り付けは、リンクフェーズにて解決されます。 最後に、 中かっこの中には、次のルールで呼び出される任意の数の *Feature* を含めることができます。
1.  最後に、 *Feature* ルール定義は次のとおり:

    ```xtext
    Feature:
        (many?='many')? name=ID ':' type=[Type];
    ```

    `many` キーワードは、複数の値を持つ特性をモデル化するのに使われます。 代入演算子 (`?=`) は、`many` の型が *boolean*であることを意味しています。 その他のパーサールールは既知のものです。

このエンティティ文法はすでにXtextの文法言語の最も重要な概念を使用しています。 キーワードは文字列リテラルとして記述され、単純な割り当てではイコール (`=`) が用いられるが、複数値の割り付けにはプラスイコール (`+=`) が用いられます。 また、真偽値の割り当て符号 (`?=`) についても確認しました。 さらに、この例では様々な構文要素 (`?` = オプション, `*` = 任意の数, `+` = 少なくとも1つ) を含んでおり、さらにクロスリファレンスのデモを示しています。 詳細は、[文法言語リファレンス](301_grammarlanguage.html) を参照してください。 では、これらの言語記述で何ができるのかを見ていきましょう。

## 言語アーティファクトの生成

文法の準備が整ったので、様々な言語要素を得るためコード生成を実行する必要があります。コード生成のためには、文法エディター上で右クリックし以下を選択します。

*Run As &rarr; Generate Xtext Artifacts*.

この操作によって、パーサー、テキストエディターといくつかの追加のインフラストラクチャコードが生成され、これはらコンソールビューのログメッセージで確認することができます。

![](../../documentation/images/30min_rungenerator.png)

## 生成されたEclipseプラグインの実行 {#run-generated-plugin}

Eclipse IDE統合の準備ができました。パッケージエクスプローラのプロジェクト `org.example.domainmodel` を右クリックし、*Run As &rarr; Eclipse Application*を選択すると、新規run configurationが生成され、新たな言語プラグインが組み込まれた2つ目のEclipseインスタンスが表示されます。表示されたEclipseインスタンスにて*File &rarr; New &rarr; Project... &rarr; Java Project*を実行し、新規プロジェクトを作成します。その後、拡張子が (*\*.dmodel*) から始まるファイルを作成すると、生成したエンティティエディターが開きます。では、コード補間、シンタックスハイライト、構文チェック、リンクエラー、フォーマッティング、(クイック)アウトラインビュー、ハイパーリンキング、参照の発見、折り畳み、リネームリファクタリングなどの標準機能を確認しましょう。

![](../../documentation/images/30min_editor.png)

## 第2イテレーション: パッケージの追加とインポート {#add-imports}

最初のDSLを生成しエディターの表示を確認したので、言語の改良、機能追加をしていきましょう。domainmodel言語は名前の衝突を避けることとJavaとの親和性を高めるために*Packages*の概念をサポートする必要があります。*Package* は *Types* と他のパッケージを含みます。さらに名前による参照を実現するため、imports宣言も追加します。

最後に、これまで使っていたモデルを異なるファイルに分割します。

```domainexample
// datatypes.dmodel

datatype String
```

```domainexample
// commons.dmodel

package my.company.common {

    entity HasAuthor {
        author: String
    }
}
```

```domainexample
// blogs.dmodel

package my.company.blog {

    import my.company.common.*

    entity Blog {
        title: String
        many posts: Post
    }

    entity Post extends my.company.common.HasAuthor {
        title: String
        content: String
        many comments: Comment
    }

    entity Comment extends HasAuthor {
        content: String
    }
}
```

文法を改良しましょう。

1.  *Domainmodel* は型とパッケージは含まれているので、エントリールールを変更する必要があります。さらに、*PackageDeclarations* と *Types* に対する上位の共通型(*AbstractElement*)も導入する必要があります。

    ```xtext
    Domainmodel:
        (elements+=AbstractElement)*;

    AbstractElement:
        PackageDeclaration | Type;
    ```

1.  `PackageDeclaration`は期待通りに見えます。これは、複数の *Imports* と *AbstractElements* を含みます。*Imports* もまたルートドメインモデルに含まれる必要があるため、`AbstractElement` のルールに追加します。

    ```xtext
    PackageDeclaration:
        'package' name=QualifiedName '{'
            (elements+=AbstractElement)*
        '}';

    AbstractElement:
        PackageDeclaration | Type | Import;
    
    QualifiedName:
        ID ('.' ID)*;
    ```

    `QualifiedName` は少し特殊で、割り当てが含まれません。したがって、これは文字列を返すデータタイプルールを提供します。よって、 *Package*の`name`特性は[String]({{site.javadoc.java}}/java/lang/String.html)型のままです。
1.  ImportsはXtextによってとても簡単に定義することができます。パーサールールで`importedNamespace`という名前を用いると、フレームワークはimportとして処理します。これはワイルドカードもサポートしています。

    ```xtext
    Import:
        'import' importedNamespace=QualifiedNameWithWildcard;

    QualifiedNameWithWildcard:
        QualifiedName '.*'?;
    ```

    `QualifiedName`と同様に、`QualifiedNameWithWildcard`もまたもプレーン文字列を返します。
1.  最後のステップでは、完全修飾名をクロスリファレンスでも使用できるようにします。これがないと、importを常に記載しないとエンティティの参照ができなくなります。

    ```xtext
    Entity:
        'entity' name=ID ('extends' superType=[Entity|QualifiedName])? '{'
            (features+=Feature)*
        '}';

    Feature:
        (many?='many')? name=ID ':' type=[Type|QualifiedName];
    ```

    (`|`)はクロスリファレンスの対象ではなく、パースされた文字列の構文を指定するために用いられることに注意してください。

文法は以上です。最終的に以下のようになります。

```xtext
grammar org.example.domainmodel.Domainmodel with
                                      org.eclipse.xtext.common.Terminals

generate domainmodel "http://www.example.org/domainmodel/Domainmodel"

Domainmodel:
    (elements+=AbstractElement)*;

PackageDeclaration:
    'package' name=QualifiedName '{'
        (elements+=AbstractElement)*
    '}';

AbstractElement:
    PackageDeclaration | Type | Import;

QualifiedName:
    ID ('.' ID)*;

Import:
    'import' importedNamespace=QualifiedNameWithWildcard;

QualifiedNameWithWildcard:
    QualifiedName '.*'?;

Type:
    DataType | Entity;

DataType:
    'datatype' name=ID;

Entity:
    'entity' name=ID ('extends' superType=[Entity|QualifiedName])? '{'
        (features+=Feature)*
    '}';

Feature:
    (many?='many')? name=ID ':' type=[Type|QualifiedName];
```

上記の変更をエディターに反映させるためには、前のセクションで紹介した言語インフラストラクチャの生成を再度行う必要があります。また、モデルを小さなパーツに分割し、ファイルの境界を越えてクロスリファレンスすることも可能です。

![](../../documentation/images/30min_multipleeditors.png)

---

**[次章: 15分チュートリアル - 拡張](103_domainmodelnextsteps.html)**
