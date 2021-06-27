---
layout: documentation
title: Webエディターサポート
part: リファレンスドキュメント
---

# {{page.title}} {#web-integration}

バージョン2.9からXtextはWebアプリケーション内のテキストエディター統合のためのインターフェースを提供しています。テキストエディターはJavaScriptにより実装されており、コード補完のような言語関連サービスはサーバーサイドコンポーネントへのHTTPリクエストにより実現されています。

## クライアント {#client}

Xtextは3つのJavaScriptテキストエディターライブラリーをサポートしています。

 * [Orion](http://eclipse.org/orion/)
 * [Ace](http://ace.c9.io)
 * [CodeMirror](http://codemirror.net)

下表に示すエディターライブラリーのどれを選ぶかによってサポートされる言語関連サービスが異なります。

<table class="table table-bordered">
  <thead>
    <tr>
      <th></th>
      <th>Orion</th>
      <th>Ace</th>
      <th>CodeMirror</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="#content-assist">コンテンツアシスト</a></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
    <tr>
      <td><a href="303_runtime_concepts.html#validation">検証</a></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
    <tr>
      <td><a href="#syntax-highlighting">シンタックスハイライト</a></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
    <tr>
      <td><a href="#semantic-highlighting">セマンティックハイライト</a></td>
      <td><div class="supported"></div></td>
      <td><div class="not-supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
    <tr>
      <td><a href="#mark-occurrences">強調表示</a></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
    <tr>
      <td><a href="#hover-info">ホバー</a></td>
      <td><div class="supported"></div></td>
      <td><div class="not-supported"></div></td>
      <td><div class="not-supported"></div></td>
    </tr>
    <tr>
      <td><a href="303_runtime_concepts.html#formatting">フォーマット</a></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
    <tr>
      <td><a href="303_runtime_concepts.html#code-generation">コード生成</a></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
    <tr>
      <td><a href="#persistence">永続化</a></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
      <td><div class="supported"></div></td>
    </tr>
  </tbody>
</table>

XtextのJavascript統合ではモジュールとその依存関係の管理のため[RequireJS](http://requirejs.org)を使います。主なモジュールは使用するエディターにより異なり `xtext/xtext-orion`、 `xtext/xtext-ace`もしくは `xtext/xtext-codemirror`のどれかです。Xtextと一緒にこれらのライブラリーをロードする例は、`generateHtmlExample = true`とした`WebIntegrationFragment`によって生成できます。([言語ジェネレーター](302_configuration.html#generator)参照)[WebJars](https://www.webjars.org/)に基づく最小の設定を以下に示します。

#### Orion

```javascript
require.config({
    paths: {
        "text": "webjars/requirejs-text/<version>/text",
        "jquery": "webjars/jquery/<version>/jquery.min",
        "xtext/xtext-orion": "xtext/<version>/xtext-orion"
    }
});
require(["orion/code_edit/built-codeEdit-amd"], function() {
    require(["xtext/xtext-orion"], function(xtext) {
        xtext.createEditor();
    });
});
```
WebJarsではOrionが入手できないため、[download.eclipse.org](http://download.eclipse.org/orion/)からダウンロードする必要があります。

#### Ace

```javascript
require.config({
    paths: {
        "jquery": "webjars/jquery/<version>/jquery.min",
        "ace/ext/language_tools": "webjars/ace/<version>/src/ext-language_tools",
        "xtext/xtext-ace": "xtext/<version>/xtext-ace"
    }
});
require(["webjars/ace/<version>/src/ace"], function() {
    require(["xtext/xtext-ace"], function(xtext) {
        xtext.createEditor();
    });
});
```

#### CodeMirror

```javascript
require.config({
    paths: {
        "jquery": "webjars/jquery/<version>/jquery.min",
        "xtext/xtext-codemirror": "xtext/<version>/xtext-codemirror"
    },
    packages: [{
        name: "codemirror",
        location: "webjars/codemirror/<version>",
        main: "lib/codemirror"
    }]
});
require(["xtext/xtext-codemirror"], function(xtext) {
    xtext.createEditor();
});
```

### JavaScript API

#### `xtext.createEditor(options)`

エディターインスタンスを1つ、それ以上作成しそれらに対してXtextサービスを初期化します。利用可能なオプションは以下で説明します。作成したエディターが1つの場合、戻り値は作成したエディターです。複数作成した場合、戻り値はエディターの配列です:

```javascript
var editor = xtext.createEditor();
```

Orionではエディターが直接戻り値になることはなく、_promise_を介して返されます:

```javascript
xtext.createEditor().done(function(editorViewer) {
    ...
});
```

エディターの親要素がオプションで指定されていない場合、この関数は`id="xtext-editor"`の要素を検索します。もし、`id="xtext-editor"`の要素がない場合、この関数は`class="xtext-editor"`の要素を検索します。

#### `xtext.createServices(editor, options)`

生成済みのエディターに対してXtextサービスを初期化します。エディターの生成を完全にコントロールしたい場合は、この変数を使ってください。

#### `xtext.removeServices(editor)`

エディターから全てのサービスと関連するリスナーを削除します。この操作はJavascriptによるシンタックスハイライトには効果がありません。

### Options

オプションは `createEditor`もしくは`createServices`関数に渡されるオブジェクトを通じて指定することができます。

```javascript
xtext.createEditor({
    resourceId: "example.statemachine",
    syntaxDefinition: "xtext/ace-mode-statemachine"
});
```

または `createEditor`を用いる場合、エディターが生成されたHTML要素内のアトリビュートにてオプションを設定することができます。アトリビュート名はJavaScriptオプション名をキャメルケースからハイフンセパレートに変換し接頭語`data-editor-`をつけてものが生成されます。例えば、`resourceId`は`data-edirot-resource-id`になります:

```html
<div id="xtext-editor"
    data-editor-resource-id="example.statemachine"
    data-editor-syntax-definition="xtext/ace-mode-statemachine"></div>
```

[利用可能なすべてのオプション](#options-reference)は、以下を参照してください。

### テキストコンテンツの取得 {#getting-text-content}

テキストコンテンツはXtextサーバーからロードされる、もしくはJavascriptにより提供されます。いずれにせよ、Xtextサーバーはリクエストを処理するために使用している言語を識別する必要があります。言語は通常ファイルの拡張子かコンテンツタイプにて識別されます。ファイル拡張子は`xtextLang`オプションにより指定可能です。一方、コンテンツタイプは`contentType`オプションで与えられます。

### サーバーからテキストをロードする

`resourceId`オプションに値を指定した場合、言語はidに含まれるファイル拡張子から識別される。この場合、`xtextLang`オプションを提供する必要はありません。`resourceId`の使用によるもう1つの効果は、サーバーが永続化層からそれぞれのリソースのロードを試みることです。Xtextサーバーはデフォルトの永続化を定義していないので、この方法を用いるためには[Persistence](#persistence)節に記載に従ってカスタマイズをする必要があります。

`resourceId`文字列はサーバー上でテキストコンテンツの識別に必要な情報を含んでいます。通常、 [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)構文が用いられますが、これは必須ではありません。

### JavaScriptを通じたテキストの提供

Xtextサーバーの永続化層は完全にオプションなので、コンテンツのフェッチに好きな方法を使うことができます。例えば、クライアントサイドで生成したり、自身のサービスを経由して要求します。エディターが生成されるDOM要素内にテキストすでに含まれている場合、このテキストは初期コンテンツとして使用されます。それ以外の場合、エディターの初期値は空白であり、選択したエディターフレームワークのAPIを使ってコンテンツを好きに変更することができます。

もしXtext付属の永続化サービスを使用せずに`resourceId`を指定したい場合、`loadFromServer`オプションを`false`に設定してください。

### オペレーションモード

XtextのWeb統合では以下に示す2つのオペレーションモードがあります。

### ステートフルモード

このモードは `sendFullText`を`false`(デフォルト設定)に設定したときに有効となります。ステートフルモードでは、エディターのテキストコンテンツが変更されるたびに更新要求がサーバーに送信されます。このアプローチでは、テキストのコピーがサーバーのセッション状態に保持され、ASTパースや検証といったXtext関連のサービスがテキストのコピーとともにキャッシュされます。これは、サーバー上でサービスが非常に高速で実行され、そして多くの要求メッセージのサイズが非常に小さいことを意味します。サービス要求の応答時間を最適化するにはこのモードを使用します。

#### ステートレスモード

このモードは`sendFullText`を`true`に設定したとき有効になります。この場合、テキストコンテンツを変更しても更新リクエストを送る必要はありませんが、他の全てのサービス要求にフルテキストコンテンツが添付されます。これは、要求のたびにテキストをパースする必要があり、メッセージサイズもテキストコンテンツのサイズに比例して大きくなることを意味します。このモードはサーバーサイドセッションと各セッションidをcookieに保存するのを避けたい場合にのみ使用します。

### シンタックスハイライト {#syntax-highlighting}

[セマンティックハイライト](#semantic-highlighting)と対照的に、シンタックスハイライトはサーバーへの接続要求なく純粋にJavascript内で計算できます。これは使っているエディターライブラリーのハイライト機能を用いて実現されます。3つの全てのライブラリーはハイライトルールを指定するためのJavaScript APIを提供しています。Xtext言語ジェネレーターの`WebIntegrationFragment`は、数値やコメントといった基礎的なトークンやキーワードを含むハイライト仕様を生成することができます。 ([言語ジェネレーター](302_configuration.html#generator)参照)もし、Xtext言語ジェネレーターが生成したものが不十分であるならば、選択したエディターライブラリーのドキュメントに従い、自身で仕様を実装することができます。

ハイライト仕様へのパスはXtextエディターの設定中に`syntaxDefinition`オプションで設定されます。もし、オプションに値を設定しなかった場合、 `'xtext-resources/{xtextLang}-syntax'` (Orion) もしくは `'xtext-resources/mode-{xtextLang}'` (Ace) 形式で`xtextLang`から構築されます。CodeMirrorシンタックスハイライトは_mode_の登録と`mode`オプションの設定により構成されます。(デフォルト名: `'xtext/{xtextLang}'`)

### サービスの実行

`createEditor` と `createServices`関数は、エディターを使用している間ほとんどのサービスを自動的に実行するためのエディターイベントに対するリスナーを設定します。また、全てのサービスは各エディターインスタンスに割り当てている`xtextServices`オブジェクトを使用することでプログラムで実行することも可能です。以下のコードはid `save-button`のボタンがクリックされたときにエディターに関連付けられているリソースを保存する例です:

```javascript
var editor = xtext.createEditor();
$("#save-button").click(function() {
    editor.xtextServices.saveResource();
});
```

採用したエディターライブラリーによって有効化そしてサポートされる各サービスは以下の関数によって提供されます。全ての関数は要求データを最終的に解決する_promise_を返します。さらに、全ての関数に`options`パラメータを与えることで、エディタービルド時に宣言したオプションの一部を上書きすることができます。

 * `getContentAssist()`
   [コンテンツアシスト](#content-assist)節で説明した形式でコンテンツアシスト案を返します。
 * `validate()`
   検証結果を返します。戻り値のオブジェクトは`issues`プロパティを持っており、これは以下のプロパティのオブジェクト配列です:
    * `description`: ユーザに表示する説明。
    * `severity`: `'error'`、 `'warning'`、 or `'info'`のいずれか
    * `line`: 問題が発生した行 (1から開始)
    * `column`: 問題が発生した列 (1から開始)
    * `offset`: ドキュメント内の文字オフセット (0から開始)
    * `length`: 影響を受けるテキスト領域の長さ
 * `computeHighlighting()`
   セマンティックハイライト用のテキストスタイリングデータを返します。 ([セマンティックハイライト](#semantic-highlighting)節を参照)。
 * `getOccurrences()`
   現在のカーソル位置にあるエレメントを返します。 ([強調表示](#mark-occurrences)節を参照)。
 * `getHoverInfo()`
   現在のカーソル位置にあるホバーを返します。 ([ホバー](#hover-info)節を参照)。
 * `format()`
   Formats the current selection (or the whole document if the selection has zero length) and returns the formatted text.
   現在の選択範囲をフォーマットし、フォーマット後のテキストを返します。 (選択範囲の長さが0の場合、全てのドキュメントをフォーマット対象とする)
 * `generate()`
   関連付けられたリソースから生成されたドキュメントを返します。 1つ以上のドキュメントが生成された場合、 `documents`プロパティを持つオブジェクトが返されます。`documents`プロパティは`name`、`contentType`と`content`プロパティを持つオブジェクトの配列です。
 * `loadResource()`
   関連付けられたリソースをロードし、`fullText`と`dirty`プロパティを持つオブジェクトを返します。現在のセッション中にリソースが変更された場合、ページがリロードされても変更後のバージョンが返されます。
 * `saveResource()`
   現在のリソースが保存されます。
 * `revertResource()`
   Reverts the resource to the last saved state and returns an object with properties `fullText` and `dirty`.
   リソースを最終保存時のものに戻し、`fullText`と`dirty`プロパティを持つオブジェクトを返します。
 * `update()`
   最後にサーバーにコミットしたバージョンのコンテンツと現在のエディター上のコンテンツとの差分を計算します。差分があった場合、サーバーの状態をリフレッシュするために更新要求を送信します。戻り値は更新されたサーバー状態の識別子である`stateId`のみを持つオブジェクトを返します。全ての要求は最後に取得したサーバー状態の識別子を含む必要があります。

### 全てのオプション {#options-reference}

 * `baseUrl`
   Xtextサービスがどこにあるかを表すパスセグメント。(デフォルト:`'/'`)`serverUrl`オプションを参照。
 * `contentAssistCharTriggers`
   コンテンツアシストサービスを呼び出すためのキー。(Orionのみ)
 * `contentAssistExcludedStyles`
   コンテンツアシストの呼び出しを除外するキー。 (Orionのみ)。
 * `contentType`
   Xtextサーバーへの要求に含むコンテンツタイプ。もしコンテンツタイプが設定されていない場合、`resourceId`オプション内のファイル拡張子が言語を判断するために使用されます。
 * `dirtyElement`
   エディターがダーティーとマーキングされたとき、ダーティー状態クラスが記載される要素。DOM要素もしくはDOM要素のidのどちらかを指定します。
 * `dirtyStatusClass`
   エディターがダーティーとマーキングされたとき、ダーティー要素の中に記載されるCSSクラス名です。(デフォルト: `'dirty'`)
 * `document`
   ドキュメント。もし指定されていない場合は、ドキュメント全体が使用されます。
 * `enableContentAssistService`
   コンテンツアシストを有効にするかどうか。 (デフォルト: `true`).
 * `enableCors`
   サービス要求にCORSを有効にするかどうか。 (default: `false`).
 * `enableFormattingAction`
   標準のキーストロークにフォーマットアクションをバインドするかどうか。<kbd>Ctrl</kbd>+<kbd>⇧</kbd>+<kbd>F</kbd> (<kbd>Ctrl</kbd>+<kbd>⇧</kbd>+<kbd>S</kbd> CodeMirror上の場合) / <kbd>⌘</kbd>+<kbd>⇧</kbd>+<kbd>F</kbd> (デフォルト: `false`)
 * `enableFormattingService`
   テキストフォーマットを有効にするかどうか。 (デフォルト: `true`).
 * `enableGeneratorService`
   コード生成を有効にするかどうか。(デフォルト: `true`)ジェネレーターにデフォルトのキーストロークがバインドされていない場合、JavaScriptコードから実行する必要があります。
 * `enableHoverService`
   マウスホバーを有効にするかどうか。(デフォルト: `true`)
 * `enableHighlightingService`
   セマンティックハイライトを有効にするかどうか。 (デフォルト: `true`). [シンタックスハイライト](#syntax-highlighting)と対照的に、 セマンティックハイライトはサーバー上で計算されます。
 * `enableOccurrencesService`
   強調表示を有効にするかどうか。(デフォルト: `true`)
 * `enableSaveAction`
   保存を標準キーストロークにバインドするかどうか。 <kbd>Ctrl</kbd>+<kbd>S</kbd> / <kbd>⌘</kbd>+<kbd>S</kbd> (デフォルト: `false`)
 * `enableValidationService`
   検証を有効にするかどうか。 (デフォルト: `true`)
 * `loadFromServer`
   サーバーからエディターコンテンツをロードするかどうか。 デフォルトは `resourceId`が設定されている場合は`true`、それ以外は`false`です。
 * `mouseHoverDelay`
   マウスオーバー後、ツールチップが表示されるまでの待ち時間(ミリ秒)(デフォルト: 500).
 * `parent`
   親要素の中にエディタを生成するかどうか。DOM要素もしくはDOM要素のidのどちらかを指定することができます。(デフォルト: `'xtext-editor'`)
 * `parentClass`
   `parent`オプションが設定されていない場合、このオプションは与えられたクラス名で要素を検索するために使用されます。 (デフォルト: `'xtext-editor'`)
 * `resourceId`
   テキストエディタ内に表示されているリソースの識別子です。このオプションは、各リソースに関して必要な情報をサーバーとやり取りするために送信されます。
 * `selectionUpdateDelay`
   選択が変更されてからXtextサービスが実行されるまでの待ち時間(ミリ秒)(デフォルト: 550)
 * `sendFullText`
   リクエスト毎に全テキストを送信するかどうか。(デフォルト: `false`)これはサーバーをステートレスモードで動作させるときに使用してください。このオプションが無効の場合、サーバー状態は定期的に更新されます。
 * `serviceUrl`
   XtextサーブレットのURL。オプションに値が設定されていない場合、`{location.protocol}//{location.host}{baseUrl}xtext-service`形式の`baseUrl`オプションを使用して構成されます。
 * `showErrorDialogs`
   ポップアップダイアログでエラーを表示するかどうか。(デフォルト: `false`)
 * `syntaxDefinition`
   構文定義を含むJSファイルのパス。このオプションを`'none'`に設定するとシンタックスハイライトを無効にできます。詳細は[シンタックスハイライト](#syntax-highlighting)節を参照。
 * `textUpdateDelay`
   テキストが変更されXtextサービスが実行されるまでの待ち時間(ミリ秒)(デフォルト: 500)
 * `xtextLang`
   言語名。(通常は言語用に構成されたファイル拡張子)このオプションが設定されていない場合、`resourceId`と`syntaxDefinition`オプションが設定されていない場合、それらを指定するために使用されます。

## サーバー {#server}

言語固有のリソースは[XtextServlet]({{site.src.xtext_web}}/org.eclipse.xtext.web.servlet/src/main/java/org/eclipse/xtext/web/servlet/XtextServlet.java)にて処理されるHTTPリクエストを通じて提供されます。このクラスは言語[インジェクター]({{site.javadoc.guice}}/com/google/inject/Injector.html) のライフサイクルの管理も担当しています。([依存性の注入](302_configuration.html#dependency-injection)参照)標準的なアプローチはサーブレットを起動したとき、インジェクターを生成し、[IResourceServiceProvider.Registry]({{site.src.xtext_core}}/org.eclipse.xtext/src/org/eclipse/xtext/resource/IResourceServiceProvider.java)に登録する方法です。デフォルトのふるまいをオーバーライドするため、あなたの言語の`<LanguageName>WebModule`もしくは`<LanguageName>IdeModule`内のバインドを変更もしくは追加することができます。

サーバーアプリケーション内にXtextサーブレットを含めるための標準的な方法は、[XtextServlet]({{site.src.xtext_web}}/org.eclipse.xtext.web.servlet/src/main/java/org/eclipse/xtext/web/servlet/XtextServlet.java)サブクラスを生成し、`init()`と`destroy()`を実行環境を管理するためオーバーライドし、そして `urlPatterns = "/xtext-service/*"`パラメータをつけた[WebServlet]({{site.javadoc.javaee}}/javax/servlet/annotation/WebServlet.html)アノテーションを追加します。[MyXtextServlet]({{site.src.xtext_web}}/org.eclipse.xtext.web.example.jetty/src/main/java/org/eclipse/xtext/web/example/jetty/MyXtextServlet.java)の例を参照してください。

Xtextサーブレットを用いずに言語固有サービスを提供するための独自の通信チャンネルを実装する場合、[XtextServiceDispatcher]({{site.src.xtext_web}}/org.eclipse.xtext.web/src/main/java/org/eclipse/xtext/web/server/XtextServiceDispatcher.java)インターフェースをインジェクトし、`getService(IServiceContext)`を呼び出すことで実現できます。引数である[IServiceContext]({{site.src.xtext_web}}/org.eclipse.xtext.web/src/main/java/org/eclipse/xtext/web/server/IServiceContext.java)は要求パラメータとクライアントセッションを提供するために実装しなければなりません。さらに、[XtextServiceDispatcher]({{site.src.xtext_web}}/org.eclipse.xtext.web/src/main/java/org/eclipse/xtext/web/server/XtextServiceDispatcher.java)をサブクラス化して、ドキュメントASTと関連する全てのXtext APIにアクセスするためのカスタムサービスを追加することもできます。

以降にWebエディタの標準サービスのカスタマイズ方法を示します。

### コンテンツアシスト {#content-assist}

クロスリファレンスのコンテンツアシスト提案は[IdeCrossrefProposalProvider]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/editor/contentassist/IdeCrossrefProposalProvider.java)によって生成され、その他の文法要素は[IdeContentProposalProvider]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/editor/contentassist/IdeContentProposalProvider.java) が使用されます。提案のカスタマイズのためには、これらのプロバイダーのサブクラスを作成し、IDEガイドモジュールにそれらを登録します。

[IdeContentProposalProvider]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/editor/contentassist/IdeContentProposalProvider.java)は文法要素の各型に対するディスパッチメソッド`_createProposals(...)`を1つもっています。多くの場合、最良の選択は[Assignments]({{site.src.xtext_core}}/org.eclipse.xtext/emf-gen/org/eclipse/xtext/Assignment.java)のメソッドをオーバーライドし、生成された言語のGrammarAccessインスタンスと比較し、正しい割り当てをフィルターすることです。[ContentAssistEntry]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/editor/contentassist/ContentAssistEntry.java)のインスタンスを作成・構成し、与えられたアクセプターに引き渡すことで提案が提示されます。ContentAssistEntryクラスはWebブラウザーに提案を表示し、ドキュメントに反映するために必要な全ての情報を格納しています。通常クライアントへはJSONフォーマットで送信されます。

[IdeCrossrefProposalProvider]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/editor/contentassist/IdeCrossrefProposalProvider.java)の典型的なカスタマイズポイントは、スコーププロバイダーによって検索された各エレメントに呼び出される`createProposal(...)`メソッドです。ここで[ContentAssistEntry]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/editor/contentassist/ContentAssistEntry.java)に入力する情報を調整することができます。

### セマンティックハイライト {#semantic-highlighting}

デフォルトのXtextエディタには、セマンティックハイライトはありません。(シンタックスハイライト、例えばキーワードと文字列は[シンタックスハイライト](#syntax-highlighting)で示したようにクライアントサイドで処理される。)スタイルをテキストに追加する場合、[DefaultSemanticHighlightingCalculator]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/editor/syntaxcoloring/DefaultSemanticHighlightingCalculator.java)のサブクラスを作成し、 `highlightElement(...)`をオーバーライドします。ここで、アクセプターにテキストオフセット、長さそしてCSSクラス名を与えることで提示されるCSSを用いてテキスト範囲をマークすることができます。Xtextエディターを含むWebページに含まれるCSSファイルにより実際のテキストスタイルを設定することができます。

### 強調表示 {#mark-occurrences}

選択された要素の強調表示サービスは[OccurrencesService]({{site.src.xtext_web}}/org.eclipse.xtext.web/src/main/java/org/eclipse/xtext/web/server/occurrences/OccurrencesService.java)によって処理されます。`filter(EObject)`をオーバーライドすることでこのサービスから一部の要素を除外することができます。実際にマークされるテキスト両機はモデル内のクロスリファレンスから自動的に計算されます。

### ホバー {#hover-info}

マウスホバーポップアップはHTMLフォーマットで生成され、2つのパーツから構成されます: タイトルと説明。

タイトルはテキストラベルと画像から構成されます。ラベルは[INameLabelProvider]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/labels/INameLabelProvider.java)によって計算され、要素の`name`プロパティが存在する場合はデフォルト値として使われます。画像は[IImageDescriptionProvider]({{site.src.xtext_core}}/org.eclipse.xtext.ide/src/org/eclipse/xtext/ide/labels/IImageDescriptionProvider.java)の実装によって決定されます。デフォルトのふるまいは、`<class>-icon`形式のCSSクラスをつけた`<div>`を生成します。このとき`<class>`は対応する要素のEClassの名前です。実際の画像はCSSファイル内のクラスにCSSプロパティの`background-image`を使用して割り当てられます。

ホバーポップアップの説明部は[IEObjectDocumentationProvider]({{site.src.xtext_core}}/org.eclipse.xtext/src/org/eclipse/xtext/documentation/IEObjectDocumentationProvider.java)にて決定されます。この部分のデフォルトコンテントはドキュメントのコメントから取得されます: もし、要素の定義の前に複数行のコメント(例 `/* ... */`)がある場合、コメントのコンテンツが説明部として使用されます。

### 永続化 {#persistence}

対応しない限り、Xtextサーバは永続化機能を提供しません。[テキストコンテントの取得](#getting-text-content)節で述べたように、Webエディタにテキストを入力する方法は複数あります。Xtextサーバーに永続化サービスを含めたい場合、[IServerResourceHandler]({{site.src.xtext_web}}/org.eclipse.xtext.web/src/main/java/org/eclipse/xtext/web/server/persistence/IServerResourceHandler.java)を実装する必要があります。インターフェースで宣言されている`get`と`put`命令は、接続したい永続化層に委任する必要があります。[FileResourceHandler]({{site.src.xtext_web}}/org.eclipse.xtext.web/src/main/java/org/eclipse/xtext/web/server/persistence/FileResourceHandler.java)に簡単な例を示しています。

---

**[次章: LSPサポート](340_lsp_support.html)**
