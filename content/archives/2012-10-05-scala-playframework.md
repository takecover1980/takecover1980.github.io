+++
title = "Scala+PlayFramework2はじめました"
slug = "scala-playframework"
date = "2012-10-05"
categories = [ "develop" ]
+++

最近Scalaが人気あるみたいですね。Twitterがバックエンドで使ってることも話題になりました。そんなScalaに興味がでてきたので、とっかかりが簡単そうなPlay Framework2と合わせて触ってみます。

ドキュメントが充実してるからわざわざ書くことはあんまりないんだけど、最初は大まかな流れが分かればいいんじゃないかと思います。

## インストール

JDK6以降が前提です。あとは特に何も言う事が無いくらい簡単。[日本語公式サイト](http://playdocja.appspot.com/)からzipファイルをダウンロードして、展開してパスを通すだけです。

今回はMacのHomebrewにパッケージがあったのでそっちでインストールします。

``` bash
brew install play
```

## アプリ作成の流れ

大まかな流れはこんな感じ。分かりやすくていい感じ。

1. playコマンドでアプリケーションを作成する
2. URLとコントローラを対応付ける
3. コントローラ(+モデル)を実装する
4. 2〜3の繰り返し

## playコマンドでアプリケーションを作成する

``` bash
play new sample
```

対話式にアプリケーション名とテンプレートの選択をします。テンプレートは下の3つから選びますが、今回はScalaを使いたいので1を選びます。

1. Scalaアプリケーション
2. Javaアプリケーション
3. 空のアプリケーション

この時点で起動できます。やり方はこんな感じ。

``` bash
play  # playコンソール起動
run   # playコンソール内でアプリケーション起動
```

`http://localhost:9000`にアクセスするとwelcome画面が表示されます。サーバ起動しっぱなしでもソース変更すれば反映されるので、止めずにこのまま進めます。

![image playframework welcome](/images/20121005-playframework-welcome.png "playframework welcome")

## URLとコントローラを対応付ける

conf/routesにルートを追加します。このURLにアクセスした時に対応したコントローラのメソッドが実行されます。

とりあえずGETで/helloを追加。Applicationクラスにメソッドを追加してみます。

conf/routes

```
# Home page
GET     /                       controllers.Application.index

# Hello
GET     /hello                  controllers.Application.hello
```

## コントローラ(+モデル)を実装する

とりあえずモデルは置いといて、Application.scalaに追記します。

Application.scala

``` scala
package controllers

import play.api._
import play.api.mvc._

object Application extends Controller {

  def index = Action {
    Ok(views.html.index("Your new application is ready."))
  }

  // 信頼と実績のアレ
  def hello = Action {
    Ok("Hello, world!")
  }

}
```

`http://localhost:9000/hello`にアクセスすると、「Hello, world!」が表示されます。

![image playframework hello](/images/20121005-playframework-hello.png "playframework hello")

とりあえず触ってみるレベルではとても簡単です。J2EEでの作り方とはずいぶん違いますね。XML地獄が無いだけでもすごくありがたい。

ScalaもPlay Framework2もいろいろ作ってみるのはこれからになりますが、なかなか楽しそうです。

## 参考ページ

- [Documentation: Home — Playframework](http://playdocja.appspot.com/documentation/2.0.3/Home)
- [Play 2.0 ドキュメント · playframework-ja/Play20 Wiki · GitHub](https://github.com/playframework-ja/Play20/wiki)
