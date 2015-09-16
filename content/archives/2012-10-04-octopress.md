+++
title = "Octopressでブログしよう"
slug = "octopress"
date = "2012-10-04"
categories = [ "octopress" ]
+++

普通のブログってやつがとても面倒なんです。

CMSにログインしたり画像アップロードしたり謎エディタで記事書いたり公開ボタン押したりバックアップするのにプラグイン入れたりとか、やってられない。

手元のエディタで記事を書いてコマンド打ったら公開。そんな感じで楽したいのでOctopressに移行します。いまさら感が半端ないけどね。

## 構成

- Octopress ブログエンジン。Markdownで記事を書いてhtmlに出力する。
- Bitbucket ソース管理。Octopressのディレクトリを管理する。
- GitHub サイトの公開場所。Octopressで出力したhtmlなどを置く。

ソースはとりあえずBitbucketのプライベートリポジトリに置くことにします。なんとなくですが。

## GitHub Pagesの準備

gitのインストール、SSH Keysの登録が終わっている前提で、[Create a New Repo](https://github.com/new)でUSERNAME.github.comの名前でリポジトリを作ります。

USERNAMEは各自置き換えてくださいね。

## Octopressインストール

まずは、USERNAME.github.comでアクセスできるようにするところまで。

``` bash
git clone git://github.com/imathis/octopress.git octopress
cd octopress
sudo gem install bundler
sudo bundle install
rake install

rake setup_github_pages
Enter the read/write url for your repository:
git@github.com:USERNAME/USERNAME.github.com.gitを入力します。

rake generate
rake deploy
```

独自ドメイン（サブドメイン）で運用したいので、ドメイン会社のサイトでDNSの設定をしておきます。

```
サブドメイン：blog
種別：CNAME
内容：USERNAME.github.com
```

OctopressにCNAMEファイルを作ります。_config.ymlも書き換えて、タイトルやURLなどちゃんとしたものに設定しておきましょう。

``` bash
echo 'blog.withweb.info' > source/CNAME
rake generate
rake deploy
```

あとは、記事をどんどん追加していくだけ。

## 記事を書いて投稿する

これだけ覚えておけばだいたいいける。記事やページの追加はコマンド打たなくてもsource/_postsの中にMarkdownのファイルを突っ込むだけでいけるみたい。

``` bash
rake new_post["title"] # 新規記事を作成
rake new_page["title"] # 新規ページを作成
rake generate          # 記事をhtmlに変換
rake preview           # プレビュー(localhost:4000)
rake deploy            # 記事を公開(GitHub Pagesにデプロイ)
```

## サイトの運用

ソースはBitbucketにするのでリモートリポジトリを追加します。

``` bash
git remote add bitbucket git@bitbucket.org:USERNAME/octopress.git
git push -u bitbucket source
```

あとはpushしていけば記事のバックアップをとっていけます。

``` bash
git add .
git commit -m 'commit message'
git push
```

## Octopressの更新

Octopress自体を更新する場合は、本家の更新を取り込んでマージします。その後、ソースとスタイルを更新してpush、デプロイします。

``` bash
git fetch octopress
git pull octopress master
bundle install
rake update_source
rake update_style
rm -rf sass.old
rm -rf source.old
git push
rake deploy
```

## 参考ページ

- [Octopressのインストールから運用管理まで - T.I.D.](http://tokkonopapa.github.com/blog/2011/12/30/octopress-on-github-and-bitbucket/)
- [GithubとOctopressでモダンな技術系ブログを作ってみる - Glide Note - グライドノート](http://blog.glidenote.com/blog/2011/11/07/install-octopress-on-github/)
