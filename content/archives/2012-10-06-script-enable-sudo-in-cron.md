+++
title = "cronの中でsudoを使う方法"
slug = "script-enable-sudo-in-cron"
date = "2012-10-06"
categories = [ "vps" ]
+++

jenkins(実行ユーザ:jenkins)のcronでsudoを使う必要があり、/etc/sudoersの設定を変えたのでメモ。

ユーザ:jenkinsをtty(コンソール)無しでsudoさせるため、こんな感じで設定しました。

``` bash
sudo /usr/sbin/visudo

#Defaults requiretty           # tty無しの場合sudoさせない(コメントアウト)
Defaults:jenkins !requiretty   # ユーザjenkinsはtty無しでsudo可能
jenkins ALL=(ALL) NOPASSWD:ALL # ユーザjenkinsはパスワード無しでsudo可能
```

## ちょっと解説

sshコマンドやcronの中でsudoを使うスクリプトを使う場合によくあるケースなんですが

``` bash
sudo: sorry, you must have a tty to run sudo
```

こんなエラーで止まることがあります。これは、/etc/sudoersの設定でtty(コンソール)とつながってないsudoが禁止されているためです。

/etc/sudoersの下の行をコメントアウトするか、!requirettyを指定するとこの設定を無効化できます。

``` bash
Defaults requiretty
```

が、クラッキング目的で進入したソフト等からsudoされないようにするのが目的なので、安易に外すのもいかがなものかと思うわけです。(効果があるかどうかは置いといて)

というわけで、今回は特定のユーザのみsudoを許可する設定を行いました。

