+++
title = "VPSにGitLabをインストール"
slug = "vps-gitlab-install"
date = "2012-11-17"
categories = [ "vps" ]
+++

プライベートなリポジトリにBitbucketを使っているんですが、なんだかんだで使いにくい。そこで、お手軽とはいかないけど人気のGitHubクローンということで、GitLab使ってみます。

[プロジェクトページの手順](https://github.com/gitlabhq/gitlabhq/blob/stable/doc/installation.md)の通りにやればいいんだけど、ちょっと手を抜いて違う手順でインストールしたのでメモ。

## 構成

- CentOS 6.2
- Ruby 1.9.3-p194
- Gitolite 3.2
- GitLab 3.0.3

## 準備

とりあえず必要そうなのを入れます。足りない物があればbundle installで怒られると思うので都度入れていけばいいと思います。

``` bash
sudo yum install libxml2-devel libxslt-devel libicu-devel sqlite-devel redis
```

Redisを起動＋自動起動。いわゆるNoSQLってやつで、使うのは今回が初めて。

``` bash
sudo chkconfig redis on
sudo service redis start
```

GitoliteとGitLabのユーザを分けずにgitユーザだけで設定します。ここから先はgitユーザで作業します。

``` bash
sudo useradd git
sudo passwd git
su - git
```

rbenv派です。GitLabに合うバージョンのRubyが導入済みの場合は読み飛ばしてください。

``` bash
git clone http://github.com/sstephenson/rbenv.git ~/.rbenv
git clone http://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

~/.bash_profile

``` bash
PATH=$HOME/.rbenv/bin:$PATH:$HOME/bin
export PATH
eval "$(rbenv init -)"
```

``` bash
rbenv install 1.9.3-p194
rbenv global 1.9.3-p194
gem install bundler
rbenv rehash
```

## Gitoliteインストール

SSHの鍵はgitlabとか分かりやすい名前にした方がいいかも。

``` bash
git config --global user.email "git@example.com"
git config --global user.name "gitadmin"
git clone git://github.com/sitaramc/gitolite
ssh-keygen -t rsa -N ""
gitolite/install
gitolite/src/gitolite setup -pk ~/.ssh/id_rsa.pub
chmod -R g+rwX repositories/
```

~/.gitolite.rc

``` bash
UMASK                       =>  0007, # 0077 -> 0007
GIT_CONFIG_KEYS             =>  '.*', # '' -> '.*'
```

~/.ssh/config

``` bash
Host localhost
    Port <SSHのポート番号>
```

``` bash
chmod 600 ~/.ssh/config
```

## GitLabインストール

SQLite大好きなのでDBはSQLiteを使います。たぶんMySQL使う人が多いだろうけど。

``` bash
git clone -b stable git://github.com/gitlabhq/gitlabhq.git
cd gitlabhq
cp config/gitlab.yml.example config/gitlab.yml
cp config/database.yml.sqlite config/database.yml
```

環境に合わせてgitlab.ymlとdatabase.ymlを編集したらセットアップ。

``` bash
bundle install --without development test mysql postgres --deployment
bundle exec rake gitlab:app:setup RAILS_ENV=production
cp ./lib/hooks/post-receive ~/.gitolite/hooks/common/post-receive
chmod 750 /home/git/.gitolite
./resque.sh
```

あとは手順どおりにWebサーバや起動スクリプトを設定すればOKです。
