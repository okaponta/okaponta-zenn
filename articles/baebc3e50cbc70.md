---
title: "zennで既に公開した記事をgithub管理にしたい時の手順"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn"]
published: true
---

もともとZennのwebエディタでいくつか記事を公開していましたが、後からgithub連携したいなと思いましたので、その方法を書いておこうと思います。

## githubのアカウント連携手順はこちら

事前に連携用のリポジトリを作成して、連携しておきます。

https://zenn.dev/zenn/articles/connect-to-github

## zenn-cliのインストール(推奨)
https://zenn.dev/zenn/articles/install-zenn-cli

## 記事のエクスポート

Zennの記事の管理から`内容をmarkdownで出力`をクリックすると、本文のテキストが表示されます。
![](/images/how-to-export-to-zenn.png)

この内容を連携したリポジトリの`articles/xxxxxxxxxxxx.md`に貼り付けることで連携できます。
`xxxxxxxxxxxx.md`は「公開中」のバッジのすぐ下の文字列をコピーすれば大丈夫です。


以上です！結構簡単に連携できてホッとしております、articles配下もディレクトリ分けれたらいいなとは思いました(たくさん記事書いている人は困りそう)
