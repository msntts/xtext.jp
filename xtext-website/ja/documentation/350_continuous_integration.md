---
layout: documentation
title: 継続的インテグレーション (with Maven)
part: Reference Documentation
upsite:
  eclipse: http://download.eclipse.org/
  xtext: http://download.eclipse.org/modeling/tmf/xtext/updates/
  emf: http://download.eclipse.org/modeling/emf/emf/updates/
  mwe: http://download.eclipse.org/modeling/emft/mwe/updates/
  xpand: http://download.eclipse.org/modeling/m2t/xpand/updates/
---

# {{page.title}} {#continuous-integration}

継続的インテグレーションを始めるときに考慮すべき2つの側面があります。1つめは、Eclipseのアップデートサイトや必要なアーティファクトの生成、全てのテストの実行を継続的にビルドをする場合、2つめは、言語とそれに対応するコード生成をあなたのアプリケーションビルドに統合する場合です。この節では、これらの両方のケースをサンプルプロジェクトに沿って議論します。このプロジェクトはクローン、検査、または [github.com/xtext/maven-xtext-example](https://github.com/xtext/maven-xtext-example)からダウンロードすることができます。

以降の節では、Mavenがどのように動作するのかの基本的な知識が必要です。もし、Mavenについて知識がないのであればMavenのチュートリアルを読んでください。

## サンプルプロジェクトの概要

プロジェクトを見ると、7つのプロジェクトが存在しているのがわかります。そのうち6つは言語とそのビルドの様々な側面を表しています。まず、言語のランタイムプロジェクト、UIプロジェクトとテストプロジェクトがあります。さらに、アップデートサイトプロジェクト、フィーチャープロジェクトとそれらの親pomが格納されたプロジェクトを必要とします。この7つのプロジェクトを`example-project`とよび、これは非常に小さいアプリケーションプロジェクトであり専用のMavenプラグインを介して言語のビルドやコード生成のトリガーをかけます。ではさっそく、言語のビルド方法についてみていきましょう。

## MavenとTychoを使ったXtext言語のビルド {#tycho-build}

Xtext言語の実行時の側面はEclipseやそのOSGIコンテナに依存していませんが、Xtext言語はOSGIバンドルの形式で開発されます。この手のビルドでは、多くの人がMavenビルド用のOSGI/P2アダプタープライグインである[Tycho](http://eclipse.org/tycho/)に頼ることになると思います。TychoはOSGIバンドルのマニュフェストから多くの情報を得ます。加えて、必要な情報は各プロジェクトのルートにあるpom.xmlファイルを介して構成されます。

### 親プロジェクト (my.mavenized.herolanguage.parent)

全てのプロジェクトはルートディレクトリにある親pomにて集約されます。もし、プロジェクトをEclipseにインポートする場合、インポートしたプロジェクトは`my.mavenized.herolanguage.parent`と呼ばれます。親pomに定義された情報は自動的に集約された子プロジェクトに継承されるので、子プロジェクトで同じ情報を何度も構築する必要はありません。ここでは、2つのプラグインを構築します:

*   Xtendコンパイラプラグインは'generate-sources'にて任意のXtendファイルに対するJavaソースコードを生成する
    
    ```xml
    <pluginManagement>
      <plugins>
        <!-- xtend-maven-plugin is in pluginManagement instead of in plugins
           so that it doesn't run before the exec-maven-plugin's *.mwe2 gen;
           this way we can list it after. 
          -->
          
        <plugin>
          <groupId>org.eclipse.xtend</groupId>
          <artifactId>xtend-maven-plugin</artifactId>
          <version>${xtext.version}</version>
          <executions>
            <execution>
              <goals>
                <goal>compile</goal>
                <goal>xtend-install-debug-info</goal>
                <goal>testCompile</goal>
                <goal>xtend-test-install-debug-info</goal>
              </goals>
            </execution>
          </executions>
          <configuration>
            <outputDirectory>xtend-gen</outputDirectory>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>
    ```

*   TychoプラグインはEclipseに適合するOSGIバンドル、フィーチャーとアップデートサイトをビルドするために、プロジェクトからEclipseプラグイン固有の構成情報をピックアップし、使用します。
    
    ```xml
    <plugins>
      <plugin>
        <groupId>org.eclipse.tycho</groupId>
        <artifactId>tycho-maven-plugin</artifactId>
        <version>${tycho-version}</version>
        <extensions>true</extensions>
      </plugin>
    </plugins>
    ```

全てのプロジェクトをビルドするためには、このpomファイルを用いてmaven buildを実行する必要があります。

### アップデートサイトプロジェクト (my.mavenized.herolanguage.updatesite)

`my.mavenized.herolanguage.updatesite`プロジェクトはアップデートサイトプロジェクトであり、pom.xmlファイルとcategory.xmlファイルのみを含みます。後者はアップデートサイトにどのフィーチャーが含まれているのかを情報を含んでいます。`category.xml`が `my.mavenized.herolanguage.sdk`プロジェクトで定義している1つのフィーチャーを指し示しているのがわかると思います。

### フィーチャープロジェクト (my.mavenized.herolanguage.sdk)

このプロジェクトは構成データのみで構成されています。このプロジェクトの中にある`feature.xml`ファイルはこのフィーチャーを含むEclipseプラグイン(バンドル)を指し示しています。

### コア言語プロジェクト (my.mavenized.herolanguage)

言語プロジェクトに対する`pom.xml`は、Xtextのコード生成に関してMavenが実行すべき情報を含んでいます。最初のプラグインは標準的なJavaプロセスを介してMWE2ファイルを呼び出します。

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>1.4.0</version>
  <executions>
    <execution>
      <id>mwe2Launcher</id>
      <phase>generate-sources</phase>
      <goals>
        <goal>java</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <mainClass>org.eclipse.emf.mwe2.launch.runtime.Mwe2Launcher</mainClass>
    <arguments>
      <argument>/${project.basedir}/src/my/mavenized/GenerateHeroLanguage.mwe2</argument>
      <argument>-p</argument>
      <argument>rootPath=/${project.basedir}/..</argument>
    </arguments>
    <classpathScope>compile</classpathScope>
    <includePluginDependencies>true</includePluginDependencies>
    <cleanupDaemonThreads>false</cleanupDaemonThreads><!-- see https://bugs.eclipse.org/bugs/show_bug.cgi?id=475098#c3 -->
  </configuration>
  <dependencies>
    <dependency>
      <groupId>org.eclipse.emf</groupId>
      <artifactId>org.eclipse.emf.mwe2.launch</artifactId>
      <version>2.12.1</version>
    </dependency>
    <dependency>
      <groupId>org.eclipse.xtext</groupId>
      <artifactId>org.eclipse.xtext.common.types</artifactId>
      <version>${xtext.version}</version>
    </dependency>
    <dependency>
      <groupId>org.eclipse.xtext</groupId>
      <artifactId>org.eclipse.xtext.xtext.generator</artifactId>
      <version>${xtext.version}</version>
    </dependency>
    <dependency>
      <groupId>org.eclipse.xtext</groupId>
      <artifactId>org.eclipse.xtext.xbase</artifactId>
      <version>${xtext.version}</version>
    </dependency>
    <dependency>
      <groupId>org.eclipse.xtext</groupId>
      <artifactId>xtext-antlr-generator</artifactId>
      <version>[2.1.1, 3)</version>
    </dependency>
  </dependencies>
</plugin>
```

2つめに使用されるプラグインは、クリーンフェーズ中に生成されたリソースを含むディレクトリーを削除します。

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-clean-plugin</artifactId>
  <version>3.1.0</version>
  <configuration>
    <filesets>
		<fileset>
			<directory>${basedir}/../my.mavenized.herolanguage/src-gen/</directory>
			<includes>
				<include>**/*</include>
			</includes>
		</fileset>
		<fileset>
			<directory>${basedir}/../my.mavenized.herolanguage.tests/src-gen/</directory>
			<includes>
				<include>**/*</include>
			</includes>
		</fileset>
		<fileset>
			<directory>${basedir}/../my.mavenized.herolanguage.ide/src-gen/</directory>
			<includes>
				<include>**/*</include>
			</includes>
		</fileset>
		<fileset>
			<directory>${basedir}/../my.mavenized.herolanguage.ui/src-gen/</directory>
			<includes>
				<include>**/*</include>
			</includes>
		</fileset>
		<fileset>
			<directory>${basedir}/../my.mavenized.herolanguage.ui.tests/src-gen/</directory>
			<includes>
				<include>**/*</include>
			</includes>
		</fileset>
		<fileset>
			<directory>${basedir}/model/generated/</directory>
		</fileset>
    </filesets>
  </configuration>
</plugin>
```

### UI言語プロジェクト (my.mavenized.herolanguage.ui)

Eclipse固有の全てのコードはここに配置されます。追加する言語、すべてのエディタ、ウィザードそして設定のUIはこのプロジェクトの中に配置されます。Mavenのビルドに関しては`pom.xml`は特別なものではありません。

### 言語プロジェクトのテスト (my.mavenized.herolanguage.tests)

テストコードをアプリケーションから分離するため、全てのユニットテストをこのプロジェクトの中に配置します。 `pom.xml`はテストのために`tycho-surefire-plugin`を含みますが、特別なことは何もありません。


## 標準的なMavenビルドへの統合 {#standalone-build}

ここまでで言語をビルドできるようになったので、アプリケーションプロジェクトのビルドに言語コンパイラーを統合していきます。これにはMavenセントラルから利用可能な専用のMavenプラグインが利用可能です。ここで、参照する `example-project`プロジェクトは標準的なJavaプロジェクトであり、Eclipseプラグイン固有の情報を含むべきでなくさらに、Tychoでビルドするべきでもないです。さっそくpom.xmlとその中のXtextプラグインを見ていきましょう。

```xml
<plugin>
	<groupId>org.eclipse.xtext</groupId>
	<artifactId>xtext-maven-plugin</artifactId>
	<version>${xtext-version}</version>
	<executions>
		<execution>
			<goals>
				<goal>generate</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<languages>
			<language>
				<setup>my.mavenized.HeroLanguageStandaloneSetup</setup>
				<outputConfigurations>
					<outputConfiguration>
						<outputDirectory>src/main/generated-sources/xtend/</outputDirectory>
					</outputConfiguration>
				</outputConfigurations>
			</language>
		</languages>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>my.mavenized.herolanguage</groupId>
			<artifactId>my.mavenized.herolanguage</artifactId>
			<version>1.0.0-SNAPSHOT</version>
		</dependency>
	</dependencies>
</plugin>
```

languageセクションに複数の言語を追加することができます。言語はデフォルトの出力構成を使用しますが、Eclipse設定するときと同様に設定を上書きすることができます。

### Maven Tychoヒント

Tychoは既存のp2リポジトリに対するプロジェクトの依存性を解決します。Tychoビルドにてターゲットとなるp2リポジトリを定義する方法は2通りあります。1つめは`pom.xml`の`<repositories>`セクションに直接リポジトリのURLを定義することです。p2リポジトリはlayout=p2でマーキングされている必要があります。2つめはEclipse [target platform files](https://wiki.eclipse.org/Tycho/Target_Platform#Target_files)を使用する方法です。このアプローチは全てのモジュールの検索時に実行されるターゲットプラットフォームの解決を1度だけにできるため、とても高速に実行できます。ターゲットプラットフォームの使用はビルド時間を非常に短縮でき、特にたくさんのモジュールを使用する大きいプロジェクトでは効果が見込めます。

p2リポジトリ依存関係の解決をさらに速くするためには、巨大な[eclipse common]({{page.upsite.eclipse}}releases/photon/)リポジトリの代わりに明確にビルドリポジトリを指定することです。以下の表はp2リポジトリにて検索できるリリースしているXtextのリポジトリURLとその依存対象です。カッコ内のバージョンは、最小のサポートバージョンです。

| Xtext         | EMF           | MWE2/MWE    | Xpand       | Eclipse     | All included in |
| ------------- | ------------- | ----------- | ----------- | ----------- | ----------- |
| [2.25.0]({{page.upsite.xtext}}releases/2.25.0/)           | [2.25.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.25) (2.20.0)     | [2.12.1]({{page.upsite.mwe}}releases/2.12.1/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.19.0]({{page.upsite.eclipse}}releases/2021-03) (4.7.3) | [2021-03]({{page.upsite.eclipse}}releases/2021-03)|
| [2.24.0]({{page.upsite.xtext}}releases/2.24.0/)           | [2.24.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.24) (2.20.0)     | [2.12.0]({{page.upsite.mwe}}releases/2.12.0/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.18.0]({{page.upsite.eclipse}}releases/2020-12) (4.7.3) | [2020-12]({{page.upsite.eclipse}}releases/2020-12)|
| [2.23.0]({{page.upsite.xtext}}releases/2.23.0/)           | [2.23.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.23) (2.20.0)     | [2.11.3]({{page.upsite.mwe}}releases/2.11.3/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.17.0]({{page.upsite.eclipse}}releases/2020-09) (4.7.3) | [2020-09]({{page.upsite.eclipse}}releases/2020-09)|
| [2.22.0]({{page.upsite.xtext}}releases/2.22.0/)           | [2.22.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.22) (2.20.0)     | [2.11.3]({{page.upsite.mwe}}releases/2.11.3/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.16.0]({{page.upsite.eclipse}}releases/2020-06) (4.7.3) | [2020-06]({{page.upsite.eclipse}}releases/2020-06)|
| [2.21.0]({{page.upsite.xtext}}releases/2.21.0/)           | [2.21.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.21) (2.20.0)     | [2.11.2]({{page.upsite.mwe}}releases/2.11.2/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.15.0]({{page.upsite.eclipse}}releases/2020-03) (4.7.3) | [2020-03]({{page.upsite.eclipse}}releases/2020-03)|
| [2.20.0]({{page.upsite.xtext}}releases/2.20.0/)           | [2.20.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.20) (2.12.0)     | [2.11.1]({{page.upsite.mwe}}releases/2.11.1/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.14.0]({{page.upsite.eclipse}}releases/2019-12) (4.7.3) | [2019-12]({{page.upsite.eclipse}}releases/2019-12)|
| [2.19.0]({{page.upsite.xtext}}releases/2.19.0/)           | [2.19.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.19) (2.12.0)     | [2.11.0]({{page.upsite.mwe}}releases/2.11.0/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.13.0]({{page.upsite.eclipse}}releases/2019-09) (4.7.3) | [2019-09]({{page.upsite.eclipse}}releases/2019-09)|
| [2.18.0]({{page.upsite.xtext}}releases/2.18.0/)           | [2.18.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.18) (2.12.0)     | [2.10.0]({{page.upsite.mwe}}releases/2.10.0/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.12.0]({{page.upsite.eclipse}}releases/2019-06) (4.7.3) | [2019-06]({{page.upsite.eclipse}}releases/2019-06)|
| [2.17.0]({{page.upsite.xtext}}releases/2.17.0/)           | [2.17.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.17) (2.12.0)     | [2.10.0]({{page.upsite.mwe}}releases/2.10.0/) (2.9.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.11.0]({{page.upsite.eclipse}}releases/2019-03) (4.7.3) | [2019-03]({{page.upsite.eclipse}}releases/2019-03)|
| [2.16.0]({{page.upsite.xtext}}releases/2.16.0/)           | [2.16.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.16) (2.12.0)     | [2.9.0]({{page.upsite.mwe}}releases/2.9.0/) (2.7.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.10.0]({{page.upsite.eclipse}}releases/2018-12/201812191000) (4.7.3) | [2018-12]({{page.upsite.eclipse}}releases/2018-12/201812191000)|
| [2.15.0]({{page.upsite.xtext}}releases/2.15.0/)           | [2.15.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.15) (2.12.0)     | [2.9.0]({{page.upsite.mwe}}releases/2.9.0/) (2.7.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.9.0]({{page.upsite.eclipse}}releases/2018-09/201809191002) (4.7.3) | [2018-09]({{page.upsite.eclipse}}releases/2018-09/201809191002)|
| [2.14.0]({{page.upsite.xtext}}releases/2.14.0/)           | [2.14.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.14) (2.12.0)     | [2.9.0]({{page.upsite.mwe}}releases/2.9.0/) (2.7.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.8.0]({{page.upsite.eclipse}}releases/photon/201806271001) (4.7.3) | [Photon]({{page.upsite.eclipse}}releases/photon/201806271001)|
| [2.13.0]({{page.upsite.xtext}}releases/2.13.0/)           | [2.13.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.13) (2.12.0)     | [2.9.0]({{page.upsite.mwe}}releases/2.9.0/) (2.7.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.7.3]({{page.upsite.eclipse}}releases/oxygen/201804111000) (3.6) | [Oxygen]({{page.upsite.eclipse}}releases/oxygen/201804111000)|
| [2.12.0]({{page.upsite.xtext}}releases/2.12.0/)           | [2.13.0]({{page.upsite.eclipse}}modeling/emf/emf/builds/release/2.13) (2.12.0)     | [2.9.0]({{page.upsite.mwe}}releases/2.9.0/) (2.7.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.7.3]({{page.upsite.eclipse}}releases/oxygen/201804111000) (3.6) | [Oxygen]({{page.upsite.eclipse}}releases/oxygen/201804111000)|
| [2.11.0]({{page.upsite.xtext}}releases/2.11.0/)           | [2.12.0]({{page.upsite.eclipse}}releases/neon/201606221000) (2.12.0)     | [2.9.0]({{page.upsite.mwe}}releases/2.9.0/) (2.7.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.6.0]({{page.upsite.eclipse}}eclipse/updates/4.6/R-4.6-201606061100) (3.6) | [Neon*]({{page.upsite.eclipse}}releases/neon/201606221000)|
| [2.10.0]({{page.upsite.xtext}}releases/2.10.0/)           | [2.12.0]({{page.upsite.eclipse}}releases/neon/201606221000) (2.12.0)     | [2.9.0]({{page.upsite.mwe}}releases/2.9.0/) (2.7.1) | [2.2.0]({{page.upsite.xpand}}releases/R201605260315) (1.4)  | [4.6.0]({{page.upsite.eclipse}}eclipse/updates/4.6/R-4.6-201606061100) (3.6) | [Neon]({{page.upsite.eclipse}}releases/neon/201606221000)|
| [2.9.1]({{page.upsite.xtext}}releases/2.9.1/) 					| [2.11.1]({{page.upsite.emf}}2.11.x/core/) (2.10.2)  	 | [2.8.3]({{page.upsite.mwe}}releases/2.8.3/) (2.7.1) | [2.1.0]({{page.upsite.xpand}}releases/R201505260349) (1.4)  | [4.5.2]({{page.upsite.eclipse}}eclipse/updates/4.5/R-4.5.2-201602121500/) (3.6) | [Mars SR2*]({{page.upsite.eclipse}}releases/mars/)|
| [2.8.4]({{page.upsite.xtext}}releases/2.8.4/) 					| [2.11.1]({{page.upsite.emf}}2.11.x/core/) (2.10.2)  	 | [2.8.1]({{page.upsite.mwe}}releases/2.8.1/) (2.7.1) | [2.1.0]({{page.upsite.xpand}}releases/R201505260349) (1.4)  | [4.5.1]({{page.upsite.eclipse}}eclipse/updates/4.5/R-4.5-201506032000/) (3.6) | [Mars SR1]({{page.upsite.eclipse}}releases/mars/)|
| [2.8.3]({{page.upsite.xtext}}releases/2.8.3/), [2.8.2]({{page.upsite.xtext}}releases/2.8.2/), [2.8.1]({{page.upsite.xtext}}releases/2.8.1/) | [2.11.0]({{page.upsite.emf}}2.11/core/R201506010402/) (2.10.2)  	 | [2.8.0]({{page.upsite.mwe}}releases/2.8.0/) (2.7.1) | [2.1.0]({{page.upsite.xpand}}releases/R201505260349) (1.4)  | [4.5.0]({{page.upsite.eclipse}}eclipse/updates/4.5/R-4.5-201506032000/) (3.6) | [Mars R]({{page.upsite.eclipse}}releases/mars/201506241002/)|
| [2.7.3]({{page.upsite.xtext}}releases/maintenance/R201411190455/) | [2.10.2]({{page.upsite.emf}}2.10.x/core/S201501230452/) (2.10) | [2.7.0]({{page.upsite.mwe}}releases/R201409021051/mwe2lang/) [1.3.4]({{page.upsite.mwe}}releases/R201409021027/mwe) (2.7.0/1.2)  | [2.0.0]({{page.upsite.xpand}}releases/R201406030414) (1.4) | [4.4.2]({{page.upsite.eclipse}}eclipse/updates/4.4/R-4.4.2-201502041700) (3.6) |[Luna SR2]({{page.upsite.eclipse}}releases/luna/201502271000/)|

Xtext 2.25.0 and Eclipse 4.19 alias 2021-03をターゲットプラットフォームとした場合の定義例は以下の通りです。

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?pde version="3.8"?>
<target name="org.xtext.myxtextlanguage.target" sequenceNumber="1">
<locations>
  <location includeAllPlatforms="false" includeConfigurePhase="false" includeMode="planner" includeSource="false" type="InstallableUnit">
    <unit id="org.eclipse.xtext.sdk.feature.group" version="0.0.0"/>
    <repository location="http://download.eclipse.org/modeling/tmf/xtext/updates/releases/2.25.0/"/>
  </location>
  <location includeAllPlatforms="false" includeConfigurePhase="false" includeMode="planner" includeSource="false" type="InstallableUnit">
    <unit id="org.eclipse.jdt.feature.group" version="0.0.0"/>
    <unit id="org.eclipse.platform.feature.group" version="0.0.0"/>
    <unit id="org.eclipse.pde.feature.group" version="0.0.0"/>
    <unit id="org.eclipse.draw2d.feature.group" version="0.0.0"/>
    <unit id="org.eclipse.emf.sdk.feature.group" version="0.0.0"/>
    <unit id="org.eclipse.xpand" version="0.0.0"/>
    <unit id="org.eclipse.xtend" version="0.0.0"/>
    <unit id="org.eclipse.xtend.typesystem.emf" version="0.0.0"/>
    <unit id="org.eclipse.emf.mwe2.launcher.feature.group" version="0.0.0"/>
    <repository location="http://download.eclipse.org/releases/2021-03/"/>
  </location>
</locations>
</target>
```
