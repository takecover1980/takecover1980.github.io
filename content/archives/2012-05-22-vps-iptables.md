+++
title = "VPSのファイアウォール設定を見直す"
slug = "vps-iptables"
date = "2012-05-22"
categories = [ "vps" ]
+++

自分しか使わないサーバとはいえ、あんまり考えずに設定をしてたので見直し。

## ポリシー

- 受信パケットは原則破棄
- 送信パケットは原則通過
- 転送パケットは原則破棄
- ローカルループバックからの受信を許可
- セッションが確立した後の受信を許可
- SYN Cookies有効
- ブロードキャスト宛のPINGに応答しない
- ICMP Redirectパケットを破棄
- Source Routedパケットを破棄
- フラグメント化されたパケットを破棄
- NetBIOS関連のパケットを破棄
- 1秒間に4回を超えるPINGを破棄
- ブロードキャスト、マルチキャスト宛のパケットを破棄
- 113番ポート(IDENT)の受信は拒否応答
- 指定サービスの受信を許可
- その他のアクセスはログ記録して破棄

ひとまず特定の国やIPはブロックしていませんが、後で手を入れていくことになると思うのでこんな感じのスクリプトにしてます。

## スクリプト

``` bash
#!/bin/bash

########################################
# 前処理
########################################

# ファイアウォール停止
/etc/rc.d/init.d/iptables stop

########################################
# 基本設定
########################################

# デフォルトルール
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

# 自ホストからのアクセスを許可
iptables -A INPUT -i lo -j ACCEPT

# セッションが確立した後は許可
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

########################################
# 攻撃対策
########################################

# SYN Cookies有効(TCP SYN Flood攻撃対策)
sysctl -w net.ipv4.tcp_syncookies=1 > /dev/null
sed -i '/net.ipv4.tcp_syncookies/d' /etc/sysctl.conf
echo "net.ipv4.tcp_syncookies=1" >> /etc/sysctl.conf

# ブロードキャストアドレス宛pingには応答しない(Smurf攻撃対策)
sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1 > /dev/null
sed -i '/net.ipv4.icmp_echo_ignore_broadcasts/d' /etc/sysctl.conf
echo "net.ipv4.icmp_echo_ignore_broadcasts=1" >> /etc/sysctl.conf

# ICMP Redirectパケットは拒否
sed -i '/net.ipv4.conf.*.accept_redirects/d' /etc/sysctl.conf
for dev in `ls /proc/sys/net/ipv4/conf/`
do
    sysctl -w net.ipv4.conf.$dev.accept_redirects=0 > /dev/null
    echo "net.ipv4.conf.$dev.accept_redirects=0" >> /etc/sysctl.conf
done

# Source Routedパケットは拒否
sed -i '/net.ipv4.conf.*.accept_source_route/d' /etc/sysctl.conf
for dev in `ls /proc/sys/net/ipv4/conf/`
do
    sysctl -w net.ipv4.conf.$dev.accept_source_route=0 > /dev/null
    echo "net.ipv4.conf.$dev.accept_source_route=0" >> /etc/sysctl.conf
done

# フラグメント化されたパケットは破棄
iptables -A INPUT -f -j LOG --log-prefix '[IPTABLES FRAGMENT] : '
iptables -A INPUT -f -j DROP

# NetBIOS関連のアクセスは破棄
iptables -A INPUT -p tcp -m multiport --dports 135,137,138,139,445 -j DROP
iptables -A INPUT -p udp -m multiport --dports 135,137,138,139,445 -j DROP
iptables -A OUTPUT -p tcp -m multiport --sports 135,137,138,139,445 -j DROP
iptables -A OUTPUT -p udp -m multiport --sports 135,137,138,139,445 -j DROP

# 1秒間に4回を超えるpingは破棄(Ping of Death攻撃対策)
iptables -N LOG_PINGDEATH
iptables -A LOG_PINGDEATH -m limit --limit 1/s --limit-burst 4 -j ACCEPT
iptables -A LOG_PINGDEATH -j LOG --log-prefix '[IPTABLES PINGDEATH] : '
iptables -A LOG_PINGDEATH -j DROP
iptables -A INPUT -p icmp --icmp-type echo-request -j LOG_PINGDEATH

# 全ホスト(ブロードキャストアドレス、マルチキャストアドレス)宛パケットは破棄
iptables -A INPUT -d 255.255.255.255 -j DROP
iptables -A INPUT -d 224.0.0.1 -j DROP

# 113番ポート(IDENT)へのアクセスには拒否応答(メールサーバ等のレスポンス低下防止)
iptables -A INPUT -p tcp --dport 113 -j REJECT --reject-with tcp-reset

########################################
# 各種サービス設定
########################################

# SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# HTTP, HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

########################################
# その他
########################################

# 上記のルールにマッチしなかったアクセスはログを記録して破棄
iptables -A INPUT -m limit --limit 1/s -j LOG --log-prefix '[IPTABLES INPUT] : '
iptables -A INPUT -j DROP
iptables -A FORWARD -m limit --limit 1/s -j LOG --log-prefix '[IPTABLES FORWARD] : '
iptables -A FORWARD -j DROP

########################################
# 後処理
########################################

# サーバー再起動時にも上記設定が有効となるようにルールを保存
/etc/rc.d/init.d/iptables save

# ファイアウォール起動
/etc/rc.d/init.d/iptables start
```

何か変なことしてたらツッコミお願いします。
