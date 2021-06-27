---
layout: documentation
title: LSPサポート
part: リファレンスドキュメント
---

# {{page.title}} {#lsp-support}

Xtextは [Language Server Protocol (LSP)](https://microsoft.github.io/language-server-protocol/)に準拠する言語サーバーの生成をサポートしています。

## はじめよう {#getting-started}

**Step 1:** 新しいXtextプロジェクトを言語サーバーサポートをつけて作成する。
![Xtext project wizard](../../documentation/images/LSP_1_Xtext_Wizard.png)

**Step 2:** [Xtext - 15 Minutes Tutorial](https://www.eclipse.org/Xtext/documentation/102_domainmodelwalkthrough.html)に従いドメインモデル言語を実装する。

**Step 3:** [LSP4E](https://projects.eclipse.org/projects/technology.lsp4e)に基づき、Eclipseでドメインモデル言語サーバーを試す。

 1. コンテントタイプを`*.dmodel`ファイルに設定する
![ドメインモデルコンテントタイプ](../../documentation/images/LSP_2_DomainmodelContentType.png)

 1. ドメインモデル言語サーバーを起動するようローンチ設定を作成する:
![ドメインモデル言語サーバーランチャー](../../documentation/images/LSP_3_DomainmodelLanguageServerLauncher.png)

 1. ドメインモデルコンテントタイプをドメインモデル言語サーバランチャーに割り付ける:
![ドメインモデル言語サーバー](../../documentation/images/LSP_4_DomainmodelLanguageServer.png)

 1. LSP does not support syntax highlighting (usually it is done on the client side). The following [TextMate](https://projects.eclipse.org/projects/technology.tm4e) json file adds syntax highlighting support for the keywords, single-line and multi-line comments of the Domainmodel language:
 ```json
{
	"name": "Domainmodel",
	"scopeName": "text.dmodel",
	"fileTypes": [
		"dmodel"
	],
	"repository": {
		"general": {
			"patterns": [
				{
					"include": "#linecomment"
				},
				{
					"include": "#blockcomment"
				},
				{
					"include": "#keyword"
				}
			]
		},
		"linecomment": {
			"name": "comment.line.double-dash.dmodel",
			"begin": "(^[ \\t]+)?(?=//)",
			"end": "(?=$)"
		},
		"blockcomment": {
			"name": "comment.block.dmodel",
			"begin": "/\\*(\\*)?(?!/)",
			"end": "\\*/"
		},
		"keyword": {
			"name": "keyword.control.mydsl",
			"match": "\\b(entity|datatype)\\b|!"

		}
	},
	"patterns": [
		{
			"include": "#general"
		}
	],
	"uuid": "8383e49a-fa0d-4bb5-827b-10e8abb294ca"
}
```
 1. Genericエディターで `*.dmodel` ファイルを開き、シンタックスハイライト、コンテントアシスト、検証、コードレンス、クイックフィックス、フォーマット ... のような言語機能を調べます :
![ドメインモデル言語機能](../../documentation/images/LSP_5_DomainmodelLanguageFeatures.png)

**Step 4:** Atom、Eclipse Che、Eclipse Theia、IntelliJ IDEA、Monaco Editor、VS Code ... に基づき、LSPクライアントを思うままに実装してください。現在サポートされているLSPクライアントは[https://langserver.org/](https://langserver.org/)の`LSP Clinetns`節にて確認することができます。

## 言語機能 {#language-features}

現在、Xtextは以下のLSP言語機能をサポートしています:

<table class="table table-bordered">
	<thead>
		<tr>
			<th><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#version_3_16_0"> LSP 3.16.0 </a> (released on 2020-12-14) <br> <a href="https://github.com/eclipse/lsp4j/blob/master/CHANGELOG.md#v0100-nov-2020"> LSP4J 0.10.0 </a>(released on 2020-11-05)</th>
			<th><a href="https://www.eclipse.org/Xtext/releasenotes.html#/releasenotes/2020/12/01/version-2-24-0"> Xtext 2.24.0 </a> <br> (released on 2020-12-01)</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#diagnostic">Diagnostic</a> (aka 検証)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_completion">補完</a> (aka コンテントアシスト)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#snippet_syntax">スニペット</a> (aka テンプレート提案)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_hover">ホバー</a></td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_signatureHelp">署名補助</a></td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_declaration">宣言へ移動</a> (aka ハイパーリンキング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_definition">定義へ移動</a> (aka ハイパーリンキング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_typeDefinition">型定義へ移動</a> (aka ハイパーリンキング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_implementation">実装へ移動</a> (aka ハイパーリンキング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_references">参照の検索</a></td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_documentHighlight">ドキュメントハイライト</a> (aka 強調表示)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_documentSymbol">ドキュメントシンボル</a> (aka モデル要素、アウトラインビュー)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_codeAction">コードアクション</a> (aka クイックアシスト、クイックフィックス)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_codeLens">コードレンス</a> (aka コードマイニング)</td>
			<td><div class="not-supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_documentLink">ドキュメントリンク</a> (aka ハイパーリンキング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_documentColor">ドキュメントカラー</a></td>
			<td><div class="not-supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_formatting">ドキュメントフォーマッティング</a> (aka フォーマッティング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_rangeFormatting">選択範囲のフォーマット</a> (aka フォーマッティング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_onTypeFormatting"> タイピング中のフォーマット</a> (aka 自動編集)</td>
			<td><div class="not-supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_rename">リネーム</a> (aka リネームリファクタリング)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_foldingRange">範囲折り畳み</a> (aka 折り畳み)</td>
			<td><div class="not-supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_selectionRange">範囲選択</a> (aka ダブルクリックテキスト選択)</td>
			<td><div class="not-supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_prepareCallHierarchy">呼び出し階層</a> </td>
			<td><div class="not-supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_semanticTokens">セマンティックトークン</a> (aka セマンティックハイライト)</td>
			<td><div class="supported"></div></td>
		</tr>
		<tr>
			<td><a href="https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_linkedEditingRange">編集範囲のリンク</a> (aka リネームリファクタリング)</td>
			<td><div class="supported"></div></td>
		</tr>
	</tbody>
</table>

## ユニットテスト
自動テストはソフトウエアの保守性と品質にとって重要です。そのため、あなたの言語サーバーに対してユニットテストを書くのを強く推奨します。Xtextは自動LSPユニットテストを実装する際に役に立つ抽象クラス[org.eclipse.xtext.testing.AbstractLanguageServerTest]({{site.src.xtext_core}}/org.eclipse.xtext.testing/src/org/eclipse/xtext/testing/AbstractLanguageServerTest.xtend)を提供しています。[org.eclipse.xtext.ide.tests.server]({{site.src.xtext_core}}/org.eclipse.xtext.ide.tests/src/org/eclipse/xtext/ide/tests/server)パッケージはおおよそすべての[言語機能](#language-features)をサポートするJUnitテストケースを含んでいます。Xtextベースの言語サーバーに自動ユニットテストを実装する方法についてのインスピレーションを得るため、それらを自由に研究してください。 


**[次章: 継続的インテグレーション](350_continuous_integration.html)**
