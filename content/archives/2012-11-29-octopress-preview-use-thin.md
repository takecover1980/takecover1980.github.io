+++
title = "Octopressのプレビューにthinを使うと速くて快適になった"
slug = "octopress-preview-use-thin"
date = "2012-11-29"
categories = [ "octopress" ]
+++

なかなかお気に入りのOctopressですが、`rake preview`が妙に遅いのと「Could not determine...」ってログが鬱陶しいのが気になってました。

どうしたものかとググってみるとけっこう簡単に対策できるみたい。これはやらねばなるまい、というわけでメモ。

## やること

どうやら遅いのも邪魔なログも原因はWEBrickらしい。WEBrickやめてthinを使うためにGemfileに追加します。

Gemfile.diff

``` diff
diff --git a/Gemfile b/Gemfile

index be2518b..2df250f 100644
--- a/Gemfile
+++ b/Gemfile
@@ -13,6 +13,7 @@ group :development do
   gem 'rb-fsevent', '~> 0.9'
   gem 'stringex', '~> 1.4.0'
   gem 'liquid', '~> 2.3.0'
+  gem 'thin'
 end

 gem 'sinatra', '~> 1.3.2'
```

thinを導入してパスを通します。

``` bash
bundle install
exec $SHELL
```

参考ページでは`thin start`で起動するかRakefile修正が必要な様ですが、いつも通りプレビューするだけでいけました。(バージョンが違うから？)

``` bash
rake preview
```

## 参考ページ

- [Octopressのプレビューにthinを使う #Octopress #Github - Qiita](http://qiita.com/items/3cd2e2c84e7dcecbc22a)
- [OctopressのRake Previewにthinを利用してプレビューを高速化する - Glide Note - グライドノート](http://blog.glidenote.com/blog/2012/10/31/thin-octopress/)
