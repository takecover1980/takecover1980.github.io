+++
title = "ブログをOctopressからHugoに移行してみた"
slug = "migrate-blog-octopress-to-hugo"
date = "2015-09-16"
categories = [ "blog", "hugo", "octopress" ]
+++

「ブログの更新が面倒」という理由でWordPressをやめてOctopressを使うようにしてました。

でも、Octopressも面倒になりました。rubyだったりpython(pygments)だったりのバージョンを合わせたり、テーマの構成がよく分からなくてちょっとした変更も辛くなってきて、、

そこで、最近人気があるらしいHugoにしてみました。動作が速くて構造がシンプルみたい。

## インストール

Golangの環境はあるので、`go get`でインストールします。Macの人はbrew、Linuxの人はyumやdpkgなんかでも入れられると思います。

``` bash
go get -v github.com/spf13/hugo
```

## 移行

記事と画像ファイルをコピーして、front matterのフォーマットをyamlからtomlに変更したくらいです。

OctopressからHugoに移行する際、日付フォーマットや画像タグの変更が必要みたいですが、私の場合は元々Hugoに合っていたので変更しなくてもOKでした。

また、テーマは[公式](https://github.com/spf13/hugoThemes/)にまとめられているみたいです。全部、個別どちらでも簡単に導入できます。今回は[hugo-zen](https://github.com/rakuishi/hugo-zen)と[hugo-zenの作者さんのサイト](http://rakuishi.com/)を参考にして、`[my_blog]/layouts`に適当に作りました。

`hugo server -w`でプレビューしながら記事やレイアウトを作って、この時点でのHugoのディレクトリはこんな感じ。

``` bash
tree -L 1 [my_blog]

[my_blog]
├── README.md
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── public
└── static
```

## ホスティング

既存のGithubのユーザーページのリポジトリ([username].github.io.git)を使います。

Octopressの時は`source`ブランチはBitbucketにしていたのですが、別に分ける必要ないかなと思ったので両方Githubにまとめます。

やり方は自信ない。あってるのかなぁ。

``` bash
# Octopressで生成していたサイトのデータを取得
git clone git@github.com:[username]/[username].github.io.git

# 全部消す
cd [username].github.io.git]
rm -rf ./*

# とりあえずREADME.mdを入れておく
touch README.md

# masterブランチをコミット
git add -A
git commit -m 'delete octopress'
git push origin master

# sourceブランチを作る
git checkout -b source

# Hugoで生成したサイトのデータをコピーしてくる
cp -a [my_blog]/* ./
rm -rf public

# sourceブランチをコミット
git add -A
git commit -m 'initial commit hugo'
git push origin source

# Hugoの公開ディレクトリをsubtreeでmasterブランチに切り出す
git subtree add --prefix=public --squash origin master
git subtree pull --prefix=public --squash origin master

# 記事生成（publicディレクトリが作られる）
hugo

# publicの内容をsource、masterブランチにコミット
git add -A
git commit -m 'subtree public directory'
git push origin source
git subtree push --prefix=public origin master
```

## 記事を書いて更新していく

以下のコマンドを実行する。

``` bash
hugo
git add -A
git commit -m 'xxxxxx'
git push origin source
git subtree push --prefix=public origin master
```

[Hugoのチュートリアル](http://gohugo.io/tutorials/github-pages-blog/)にデプロイスクリプトがあったのでそれを元に作る。

``` bash
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo

# Add changes to git.
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin source
git subtree push --prefix=public origin master
```

## 参考ページ

- [Introduction to Hugo](https://gohugo.io/overview/introduction/)
- [ブログをOctopressからHugoに移行した | Unresolved](http://yet.unresolved.xyz/blog/2015/01/04/migrate-blog-to-hugo-from-octopress/)
- [OctopressからHugoへ移行した | SOTA](http://deeeet.com/writing/2014/12/25/hugo/)
- [Hugo Zen: これから Hugo を始める人向けのミニマムなテーマを作りました - rakuishi.com](http://rakuishi.com/archives/hugo-zen/)
- [Hugo で github にブログを立ち上げる Part 2 - Syati.info](http://blog.syati.info/post/create_hugo_2/)
