+++
title = "さくらのVPS 2G(CentOS6)で最初にやったことメモ"
slug = "sakura-vps2g-centos6"
date = "2012-05-20"
categories = [ "vps" ]
+++

さくらのVPS 2Gを借りたので、最初にやったことをメモ。どこかに書かないと忘れちゃう。

## ユーザ

まずはrootのパスワードを変更。

``` bash
passwd
```

直接rootでログインしたくないので一般ユーザをwheelグループで作る。

``` bash
useradd <ユーザID> -g wheel
passwd <ユーザID>
```

wheelグループをパスワードなしでsudoできるようにする。

``` bash
visudo

%wheel ALL=(ALL) NOPASSWD:ALL
```

## SSH

一般ユーザの公開鍵を登録。authorized_keysの中身はローカルPCで作った公開鍵からコピペ。

``` bash
cd /home/<ユーザID>
mkdir .ssh
chmod 700 .ssh
vi .ssh/authorized_keys

chmod 600 .ssh/authorized_keys
chown -R <ユーザID>:wheel .ssh
```

SSHの設定。ポート番号を変えて、rootログイン禁止、パスワード認証を止めて鍵交換方式で認証する。

``` bash
vi /etc/ssh/sshd_config

Port <SSHのポート番号>
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
```

ひと通り設定したらサービスを再起動。

``` bash
service sshd restart
```

## ファイアウォール

とりあえずHTTPとSSHを通す。色々なサイトで紹介されているものをベースにしてます。

``` bash
vi /etc/sysconfig/iptables

*filter
:INPUT   ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT  ACCEPT [0:0]
:RH-Firewall-1-INPUT - [0:0]
-A INPUT -j RH-Firewall-1-INPUT
-A FORWARD -j RH-Firewall-1-INPUT
-A RH-Firewall-1-INPUT -i lo -j ACCEPT
-A RH-Firewall-1-INPUT -p icmp --icmp-type any -j ACCEPT
-A RH-Firewall-1-INPUT -p 50 -j ACCEPT
-A RH-Firewall-1-INPUT -p 51 -j ACCEPT
-A RH-Firewall-1-INPUT -p udp --dport 5353 -d 224.0.0.251 -j ACCEPT
-A RH-Firewall-1-INPUT -p udp -m udp --dport 631 -j ACCEPT
-A RH-Firewall-1-INPUT -p tcp -m tcp --dport 631 -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
# HTTP, SSH
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80    -j ACCEPT
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport <SSHのポート番号> -j ACCEPT
-A RH-Firewall-1-INPUT -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

## 不要なサービスの停止

わざわざ止めるほどサービスが起動していなかったけど、いくつか止めておいた。

``` bash
chkconfig lvm2-monitor off
chkconfig yum-updateonboot off
```

## メモリ使用量を確認

``` bash
free

	             total       used       free     shared    buffers     cached
	Mem:       2054808     139612    1915196          0       8312      57288
	-/+ buffers/cache:      74012    1980796
	Swap:      2096472          0    2096472
```

メモリ使用量は140MBくらい。素の状態ならこんなもんでしょう。

## まとめ

ひとまず使い始める前の設定は最低限できたかな。ログ監視くらいはこの時点で入れておけばよかったかも。
