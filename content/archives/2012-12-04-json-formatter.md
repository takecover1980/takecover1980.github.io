+++
title = "JSONフォーマッタを公開してみた"
slug = "json-formatter"
date = "2012-12-04"
categories = [ "webservice" ]
+++

ローカルのツールを外でも使えるように公開してみました。

- [JSON formatter](http://jsonformatter.withweb.info)

![image jsonformatter.withweb.info](/images/20121204-jsonformatter-withweb-info.png "jsonformatter.withweb.info")

jQueryを使った簡単なJSONフォーマッタです。（エラー処理とか細かい事はほとんどしてませんが...）

## 作ろうとした理由

以前Yahoo!オークションAPIで遊ぼうとした時に、日本語がエンコードされたままだと中身が見づらかったんです。

さらっとデータの中身を見たかったので、JSONPを返してくるサービスならJavascriptをそのまま展開しちゃえばいいや。ということで作りました。

ついでにJSONを貼り付けたら適当にフォーマットして読みやすくしようかな、と。フォーマッタがおまけです。

## 使い方

リンク先のAboutに書いてあるんですが、[JSON Data]か[JSONP Data URL]に入力して[Process]ボタンを押すと結果が下に表示されます。

実行結果は[Close]ボタンで消したり、[Collapse]ボタンでたためます。実行結果が積まれていくのでパターン別に何度も実行するような使い方に便利だと思います。

ただ、実行結果をクリップボードにコピーしたり、バリデーションする機能はありません。
