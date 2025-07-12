---
title: "青いベンチ診断の結果をひたすらtwitterに投稿するアカウントをつくった"
emoji: "🤷‍♂️"
type: "tech"
topics:
  - "aws"
  - "twitter"
  - "lambda"
published: true
published_at: "2021-12-28 20:38"
---

[Qiita](https://qiita.com/okaponta_/items/bc2d3da0bc25ba74dc90)にも投稿したので、好きな方でお読みいただければと思います。

# はじめに
みなさま「青いベンチ」という曲はご存知でしょうか。
「この声が枯れるくらいに　君を好きと言えばよかった」からはじまるサビは聞いてて心地いいですね。

さて、私がこの曲を知ったのは[青いベンチ診断](https://shindanmaker.com/240064)というものです。
これがまた面白くて、1/157464の確率で「青いベンチ／サスケ」という文字列が揃うものになります。
確率からしてだいたい外れるんですが、以下のような診断結果が出力されてクスってきてしまいます。

https://twitter.com/okaponta_/status/1475678970440216578

揃わなかった文字列を言い合うみたいなtogetterもありました。

https://togetter.com/li/452165

2021年の終わりになぜか「ひたすらこの診断結果を投稿するBotを作りたい」と強く思ってしまったために作ることにしました。
ただ、診断メーカーにはAPIが無いようで、結果が日替わりという制約があったのでロジックを再現して投稿する仕組みを作ります。

大きく以下の流れで進めました。

 - step1 ロジック解析
 - step2 ローカルで青いベンチ診断ができるようになる
 - step3 twitterアカウントを開設し、青いベンチ診断の結果を投稿できるようになる
 - step4 1時間に1回定期実行できるようになる

# step1 ロジック解析

以下3つからロジックを推測しました。

 - 診断メーカーの作成画面
 - 結果パターンの組み合わせ数
 - [青いベンチ診断の検索結果](https://twitter.com/search?q=https%3A%2F%2Fshindanmaker.com%2F240064&src=typed_query&f=live)

まず、診断メーカーの作成方法ですが、
`[LIST1][LIST2]`のように変数を設定し、`LIST1`がとりうる値を一覧でもっておきランダムで判定するというものでした。

例えば、
`[LIST1]と[LIST2]`という出力結果で、
`[LIST1]`の候補が`a`と`b`、
`[LIST2]`の候補が`c`と`d`であれば、
出力結果は以下4通りになります。

 - `aとc`
 - `aとd`
 - `bとc`
 - `bとd`

次に、結果パターンの組み合わせ数です。
以下の画像からも分かる通り157464通りです。

![](https://storage.googleapis.com/zenn-user-upload/c61160e3a296-20211228.png)

素因数分解すると$157464=2^3 × 3^9$です。

あとは診断結果からロジックを矛盾なく組み立てていきます。(これがめちゃくちゃ大変でした)
ロジックは以下で間違いないと思います。
`[LIST1][LIST1]／[LIST1]`

 - LIST1 (候補54個)

スプレッドシートに転記してにらめっこしながら解明しました。
てっきり6×6×6×9×9×9なのかなと思ってたのですが、おそろしいほどマニュアルでした・・・

`／`の後の「サスケ」の部分が「サ」と「スケ」に分かれると思っていたのですが、50回ほど診断してみたら違和感に気づきました。
 - 「スベ」からはじまるのは全て「スベスベ」「スベチン」のいづれかとなり不自然
 - 「ケンサ」「チンベス」などどう分割してもきれいにならないワードがある

というので気づきました。**54種類も考えた作者の狂気を感じる・・・**
その後、「青いベンチ」になるであろう文字列の部分も見てみると、見慣れた文字列があって、ロジックが解明できました・・・！！！たしかに「青い」も「ベンチ」もふくまれている！

あとは診断をやりまくって54個の候補を確定させました。(3時間くらいかかった)

![](https://storage.googleapis.com/zenn-user-upload/93fc2b904778-20211228.png)

✅ Step1 完了！

# step2 ローカルで青いベンチ診断ができるようになる

実装するだけなので特に書くことはないですが、以下のようにローカルで手軽に実行できるようになりました。
ロジックはわかってしまえば単純なので、基本的にさっきのスプレッドシートをテキストエディタでいい感じにコードになるように置換してちょっと書いたくらいです。

![](https://storage.googleapis.com/zenn-user-upload/c47d548b2ff0-20211228.png)
**たくさん実行できて楽しい！！！**

goの環境がある方は、
```
go install github.com/okaponta/bluebench-cli@latest
```
でインストールでき、`bluebench-cli`と打つだけで実行できます。ぜひ実行してみてください。

また、goの環境が無い方向けにバイナリも配布しているので、
[リリースページ](https://github.com/okaponta/bluebench-cli/releases)からダウンロードして使ってください。

また、homebrewでも使えるようになってます(M1Mac限定です・・・)

```
brew tap okaponta/bluebench-cli
brew install okaponta/bluebench-cli/bluebench-cli
```

✅ Step2 完了！

# step3 twitterアカウントを開設し、青いベンチ診断の結果を投稿できるようになる

まずはメールアドレスを用意し、twitterアカウントを作成します。
https://twitter.com/blue__bench

その後、Developer PlatformからAppを追加して、Read & Write権限のTokenを発行します。
https://developer.twitter.com/en/portal/dashboard

最近UIが変わったのか、Token発行まわりでだいぶ手間取ってしまいましたが、以前自分で作った[サンプル実装](https://github.com/okaponta/tweeter/blob/main/main.go)を参考にしながら無事投稿できました。

https://twitter.com/blue__bench/status/1475745677691228162

✅ Step3 完了！

# step4 1時間に1回定期実行できるようになる

ローカルでcronを動かすのも嫌なので、以下を参考にEventBridgeからLambdaをキックするように作ってみたいと思います。
仕事ではAWS使っているのですが、今回このためだけに個人アカウントを作成しました。

https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/golang-handler.html

step3で作った`main.go`を以下コマンドで固めて、S3にアップロードします。

```zsh
GOOS=linux go build -o main
zip main.zip ./main
```

ランタイム設定のハンドラの部分を`main`に書き換えて保存します。

![](https://storage.googleapis.com/zenn-user-upload/aa4ca88e5758-20211228.png)

ここで、M1 Macの落とし穴があって、`lambda`でgoを実行する際、arm64をサポートしてないみたいで、`GOARCH=amd64`オプションを加えて再度ビルドしてアップロードしました。

```
GOARCH=amd64 GOOS=linux go build -o main
zip main.zip ./main
```

そして、Lambdaのテスト投稿から無事出力されました・・・！！！

https://twitter.com/blue__bench/status/1475756133117997057

あとは、定期的に実行させればいいので、EventBridgeからルールを登録します。
イベントスケジュールにcronで以下を指定して、1時間おきに実行するようにしました。
`0 * * * ? *`

ターゲットを先ほど作成したLambdaに向けてあげれば完成です・・・・！！！

https://twitter.com/blue__bench/status/1475783686436302851

やったぁぁぁぁぁあああああ！！！！！
この声が枯れるくらいに〜〜〜君を好きと言えばよかった〜〜〜！！！！！！

✅ Step4 完了！

# 感想

結構軽い気持ちで手をつけたのですが、いろんなことに詰まりました。。。
大きくは以下で時間を使いました。

- 謎のロジック解明(とても楽しかった)
- twitterの認証(Read & Writeトークンの付与に苦戦した)
- lambdaからgoを実行するようにする部分
- homebrew-tapで`bluebench-cli`をインストールさせるようにした部分

結構一人ではじめてのことやるとそれなりに詰まるなぁという印象でした。
会社のチームメイトのありがたさを改めて実感しました笑。

よければフォローしてあげてください。🙇
https://twitter.com/blue__bench

最後まで読んでいただきありがとうございました！！！
