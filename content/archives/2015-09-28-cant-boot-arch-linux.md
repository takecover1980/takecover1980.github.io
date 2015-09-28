+++
title = "Arch Linuxが起動しない"
slug = "cant-boot-arch-linux"
date = "2015-09-28"
categories = [ "linux" ]
+++

Arch Linuxが起動しなくなって、復旧作業をしたのでメモ。

## 現象

起動時にGRUBでカーネル選択後にエラーが表示されて止まりました。
画面の写真を撮ったりしていないのでざっくりですが、以下のような内容が表示されました。

``` bash
ERROR: Unable to find root device 'UUID-xxxxxxxxxx'
sh: can't access tty; job control turned off
[rootfs /]#
```

ディスクが見つからない。とかそんな意味みたい。

## 直前の操作

1. OSアップデート
2. シャットダウン
3. マザーボードのBIOSアップデート
4. マザーボードのBIOS設定変更(Intel VT-xを有効)
5. OS起動(ここでエラー)

一気にやったからどれが原因か分からないです。
ググってみたら[Archのフォーラム](https://bbs.archlinux.org/viewtopic.php?id=142052)に似たようなのがあったので、多分OSアップデートが原因なのでしょう。

同じ現象の人が他にもいるかなと思って探してみると、ちょいちょい見つかります。今回初めて経験したけどよくある事なのかな。

## 対処方法

作業自体は[こちら](http://archlinux-blogger.blogspot.jp/2014/11/arch-linux-cant-boot-arch-linux.html)の対応そのままでいけました。

やることは以下の4つ。Arch Linuxをインストールしている人には簡単だと思います。

- LiveCDやブータブルUSBみたいにシステム外からブートできるものを用意する
- udevを再インストールする
- mkinitcpioを再インストールする
- カーネルイメージを作りなおす

# 参考ページ

- [普段使いのArch Linux: Arch Linuxが起動しない (can't boot Arch Linux)](http://archlinux-blogger.blogspot.jp/2014/11/arch-linux-cant-boot-arch-linux.html)
- [ERROR: Unable to find root device 'xxxxxx'で起動しなくなった時の対処法 | とさいぬの隠し部屋](http://blog.myon.info/blog/2013-07-27/entry/)

