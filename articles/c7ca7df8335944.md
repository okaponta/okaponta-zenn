---
title: "terraformの仕組みざっくり解説"
emoji: "🔰"
type: "tech"
topics:
  - "terraform"
published: true
published_at: "2023-06-08 21:48"
---

terraformを業務で使用しており、なんとなく動かしてはいるけど動いている仕組みについて理解できていなかったので調べてみました

## 想定読者

- 既存のterraformを編集して、planとapplyは行っているが、自分で構築したことはない
- terraformの動作原理について知りたい

## terraform is 何？

詳細な説明はいろんな記事があるので割愛しますが、一言でいうと
**「IaCを用いてインフラ構成をコードで管理するツール」**
です

OSSでソースコードもみることができます
https://github.com/hashicorp/terraform 

## terraformよく使うコマンドの詳細

### terraform init

初期化を行います。

`.terraform`というディレクトリが作られ、プロバイダープラグインとモジュールをキャッシュします。
`tfstate`がローカルで保持している場合は`tfstate`本体、S3においている場合はその場所へのリンクが記載された`terraform.tfstate`が作成される

`tfstate`が何かはあとで解説しますが、terraformが管理しているインフラの状態を保持しておくファイルと捉えてください。

プロバイダープラグインは以下に一覧がありますが、各リソースを`terraform`で管理するためのプラグインというイメージでよいと思います。
(AWS用のプラグイン、Azure用のプラグインなどなど・・・)

https://registry.terraform.io/browse/providers 

### terraform plan

`.tfstate`との差分を検知して、必要な修正を出力してくれます。

変化した内容によって、再作成が必要なインスタンスはdelete → createの動きになるし、In Placeで変更が可能なものはIn Placeで変更するように`terraform`で判断してくれます。

### terraform apply

`terraform plan`ででてきた内容を適用します。
たまに`plan`でエラーなかったけど`apply`できないなんてこともあるので注意

### terraform destroy

applyの逆、管理しているリソースを全部削除する操作です。

## tfstate

json形式で書かれている、`terraform`が管理するリソースの状態や依存関係が書かれているファイルです。

デフォルトではローカルの `.terraform`配下にありますが、設定するとS3においてPJ内で共有できます。
ロック用のファイルをDynamoDBで管理して、競合して編集できないように設定ができます。

こんな感じ
```terraform
terraform {
  backend "s3" {
    bucket         = "hogehoge"
    key            = "hoge/fuga/terraform.tfstate"
    region         = "ap-northeast-1"
    dynamodb_table = "hoge-lock"
  }
}
```

`tfstate`の中身は以下のようなコマンドを用いて見ることができます。

- terraform graph
  - `terraform graph | dot -Tsvg > graph.svg`とかでリソースの依存関係が`svg`で出力できる
- terraform show
- terraform list

また、既存のリソースを`tfstate`管理下においたり、はずしたり、移動させたりということができます。

- terraform import
  - リソースをterraform管理下におく
- terraform mv
  - terraformのコードでの管理名称を変えたい時などに使用
- terraform rm
  - リソースをterraform管理下からはずす

## Resource と Data Source

Resourceは文字通りリソースのこと。
以下のように宣言すると、書かれたConfigurationでEC2が作成されます。

```terraform
resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

例は以下の公式チュートリアルからもってきました。
https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build

Data Sourceは外部データを参照する仕組みです。
以下のように記述すると該当の`ami`を`example`という名前で参照できるようになります。

他にも、サブネットとかをData Sourceで書いて使用したりなど色々便利な使い方があります。
直接arnとかを書かずに名前解決できるのが嬉しいポイントですね。

```terraform
data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```

こちらの例も公式ドキュメントより

https://developer.hashicorp.com/terraform/language/data-sources#using-data-sources

## モジュール

最後に、モジュールについて書きます。
実際に`terraform plan`などを実行するディレクトリをルートモジュールと呼びます。

ルートモジュールから子モジュールを以下のように呼び出すことができます。
./app-clusterのディレクトリにあるリソースを`servers = 5`という引数を与えて作成しています。

```terraform
module "servers" {
  source = "./app-cluster"

  servers = 5
}
```

モジュールを使うと嬉しい点は、複数環境で少しだけ変数の違うものを共通化できるという点です。
(本番と検品環境で同じリソースなんだけど、VPCなどなどのパラメータのみが違う場合など)

この例も公式ドキュメントより引用しています。

https://developer.hashicorp.com/terraform/language/modules/syntax

他にも`source`にはterraformのレジストリ、github、bitbucketなど指定方法は色々あるみたいです。

----

以上です！
あんまり`tfstate`まわりは意識せずに使っていたので、今回改めて調べてみてイメージ持てました。あと、公式ドキュメントがだいぶ充実していることに気づきました。今後はきちんと読んでいきたい😁

最後まで読んでいただきありがとうございます！
