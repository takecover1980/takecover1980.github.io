+++
title = "VPSにログ監視ツールlogwatchをインストール"
slug = "vps-logwatch"
date = "2012-11-13"
categories = [ "vps" ]
+++

Linuxサーバのログを整形して管理者(root)に1日1回メール送信してくれるツールです。

メール送信なのでMTAが必要です。既に設定済の場合は読み飛ばしてください。

今回は個人的なサーバなので正直sSMTPで十分ですが、最近のLinuxならPostfixが標準だと思うのでそっちを使います。また、ドメインは信頼と実績のexample.comなので適時読み替えてください。外部からの受信、転送はしないのでDNSは触りません。

## インストール

yumでサクッとインストール。

``` bash
sudo yum install -y postfix logwatch
```

## Postfixの設定

ローカルと外向きにメール送信できるように以下を編集します。メールボックスはMaildir形式。

/etc/postfix/main.cf

``` bash
myhostname = mail.example.com                                           // 自FQDN名
mydomain = example.com                                                  // 自ドメイン名
myorigin = $mydomain                                                    // 送信元ドメイン名
inet_interfaces = localhost                                             // 待ち受けるネットワークインタフェース
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain  // ローカルで受信するドメイン名
home_mailbox = Maildir/                                                 // メールボックス形式
smtpd_banner = $myhostname ESMTP unknown                                // バージョン非表示
```

Maildir形式のメールボックスを使うので、新規ユーザ用にスケルトンを作っておく。既存ユーザのメールボックスを移行する場合はググればすぐ見つかります。

``` bash
sudo mkdir -p /etc/skel/Maildir/{new,cur,tmp}
sudo chmod -R 700 /etc/skel/Maildir/
```

メール送信できるか確認

``` bash
echo "test message." | mail -s test <受信確認用の外部のメールアドレス>
echo "test message." | mail -s test <受信確認用のローカルユーザ>
```

外部のメールアドレスにメールが届くのと、ローカルユーザの$HOME/Maildir/newにメールが届いていることを確認します。

## logwatchの設定

と言っても、今回はlogwatch自体は設定しません。ログレベルやアーカイブ有無など設定したい場合は/etc/logwatch/conf/logwatch.confを編集します。

設定するのはこっち。logwatchはデフォルトでroot宛てのメール転送先を設定します。

/etc/aliases

``` bash
root: example@example.com
```

logwatchをインストールすると/etc/cron.daily/00logwatchも作られるので、ほっといても1日1回メール送信されます。が、コマンドでメール送信できるので確認しておきます。

``` bash
logwatch --mailto root
```

とりあえずこれで簡単にログ監視できます。
