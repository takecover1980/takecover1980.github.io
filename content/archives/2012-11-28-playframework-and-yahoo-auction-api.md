+++
title = "PlayFramework2とYahoo!オークションAPIを使ってみた"
slug = "playframework-and-yahoo-auction-api"
date = "2012-11-28"
categories = [ "webservice", "hogehoge" ]
+++

Yahoo!オークションAPIを使ってみたくなったので、とりあえず動くものを作ってみました。

- [いまYahoo!オークションで人気の商品はコレ！](http://auction.drive7.info)

![image auction.drive7.info](/images/20121128-auction-drive7-info.png "auction.drive7.info")

いかにもアフィリエイトっぽくてアレな感じのタイトルですね。実際アフィリエイトID使っています。

適当に用意したカテゴリから検索した結果をパネルっぽく表示して、クリックするとオークションのページに飛べます。

## 準備

だいたいの流れはYahoo!デベロッパーネットワークの[ご利用ガイド](http://developer.yahoo.co.jp/start/)に手順が書いてありますが、他に必要になりそうなドキュメントはこのへんです。

1. [APIドキュメント](http://developer.yahoo.co.jp/webapi/auctions/)
2. [サンプルコード](http://developer.yahoo.co.jp/sample/auctions/)
3. [クレジット表示](http://developer.yahoo.co.jp/attribution/)

2のサンプルコードのページにサンプルリクエストURLが載っているので、ブラウザのURLバーに入れる等で試せます。

アフィリエイトIDの確認と使い方は[アフィリエイトプログラム](http://developer.yahoo.co.jp/appendix/auctions/affiliate.html)に書いてあります。Yahoo!ポイントで受け取るか、バリューコマースを使うか選べるようです。

## APIを使う

JavascriptにアプリケーションIDを直書きするのはよくないかなぁと思ったので、Webサービスを作ります。

それだけならYahoo! Pipesで十分だと思いますが、今回は使いたいのでPlay使います。おおまかにはこんな感じです。

### Playframework2

conf/routes

```
GET /yapi controllers.YahooAPI.auction(q: String ?= "book")
```

controllers/YahooAPI.scala

``` scala
package controllers

import play.api._
import play.api.mvc._
import play.api.libs._
import play.api.libs.json._
import play.api.libs.ws._

object YahooAPI extends Controller {
  def auction(q: String) = Action { request =>
    val appid = "YOUR_APPID"
    val baseurl = "http://auctions.yahooapis.jp/AuctionWebService/V2/json/search"
    val url = baseurl + "?appid=" + appid + "&query=" + q

    Async {
      WS.url(url).get().map { response =>
        Ok(response.body).as("application/json")
      }
    }
  }
```

`http://example.com/yapi?q=book`にアクセスするとbookの検索結果がJSONで返ってきます。

### jQuery

script.js

``` javascript
// API呼び出し
$.ajax({
  url: "http://example.com/yapi",
  data: {
    "q":"ほげほげ"
  },
  dataType: "jsonp",
  complete: function() {
    // 終了時の処理
  }
});

// コールバック
function loaded(json) {
  // API呼び出しの結果をゴニョゴニョする
}
```

取得した商品情報をレイアウトすれば完成です。

いろいろ手を抜いてます。アクセスが多いならYahooのサーバに優しくしておいた方がいいと思います。リクエストをキューでコントロールするとか、同一クエリは数秒キャッシュするとか、複数のアプリケーションIDでローテーションするとか。あ、最後のは違うか。
