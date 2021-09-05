Websiteをテストする方法
=======================

このwebsiteは[Jekyll](http://jekyllrb.com)を用いて生成しています。
```
gem install bundler jekyll jekyll-markdown-block
```
websiteを生成するには以下を実行してください。
```
  jekyll build --unpublished
```
実行後、`_site`フォルダにwebsiteが生成されます。websiteは以下のコマンドを実行することで、ローカルでテスト可能です。
```
  jekyll serve --unpublished
```
websiteは[server](http://127.0.0.1:4000) で確認できます。websiteはファイルを更新すると自動的に変更が反映されます。


Ruby トラブルシューティング
--------------------

- Linux/Mac OSの場合、Rubyのインストール管理は[RVM](https://rvm.io/) を使用することを推奨しています。
- Windowsの場合、[development kit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit)をインストールしてください。

### hudson用設定:

```
rvm autolibs disable
rvm install ruby-2.2.0
rvm use 2.2.0
~/.rvm/rubies/ruby-2.2.0/bin/gem install jekyll
~/.rvm/rubies/ruby-2.2.0/bin/gem install bundler
~/.rvm/rubies/ruby-2.2.0/bin/gem install jekyll-markdown-block
```
