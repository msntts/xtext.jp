翻訳環境の構築
=======================
基本的にdocker-composeを実行するだけです。本コンテナは、環境を提供するだけのものです。

**xtext-websiteの翻訳**
```
sudo docker-compose-xtext up -d
```
**xtend-websiteの翻訳**
```
sudo docker-compose-xtend up -d
```

それぞれのコンテナは、本リポジトリのxtext-websiteもしくはxtend-websiteをマウントしているので、ホストにて翻訳ファイルの編集自体が可能です。

また、コンテナ作成時はコンテナにて以下のコマンドを実行してください。
```
bundle install
```
翻訳ファイル(.poファイル)の反映は、コンテナにて以下のコマンドを実行してください。
```
rake
```