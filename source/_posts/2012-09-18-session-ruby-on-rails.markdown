---
layout: post
title: "rubykaigi2012 Ruby on Rails 関連セッション"
date: 2012-09-18 12:00
comments: true
categories: rails インフラ
---

[札幌rubykaigi2012](https://github.com/sprk2012/sprk2012-cfp)

#Rails3 レシピブック外伝#

[動画](http://www.ustream.tv/recorded/25419544)


#RubyやLinux対応を積極的に進めるマイクロソフトのオープンソース<3(LOVE)戦略#

[動画](http://www.ustream.tv/recorded/25394761)

Windows 64bit用に、artonさん作のプラットフォーム「能楽堂」の紹介がある。イチから作り、IISとオサラバしたとのこと。


#Squaleの裏側#

SqualeというPAASの内部構造
[動画](http://www.ustream.tv/recorded/25395947)

- Ruby Sinatra Rails publish & deploy.
- サーバーセットアップを忘れてコードに専念！
- ライトユーザーの敷居を下げるのが目標

Amazon EC2 instance上にサーバー群を構成している。
EC2 Instance を仮想コンテナで区切っている。
(メモリを多く使うので、M2 large などを使う。）


##構成##

- Nginx
- Unicorn
- SSHD
- Supervisord
- paperboy-sqale


##Containers##

- made by LXC
- 仮想環境で動く（Heroku Dynos と同じ）
- 監視は、プロセスの動作しか見ていない（今後、port監視など強化したい。）


##セキュリティ##

- tcp port bind を制限し、一般Userからは仕掛けられなくする。
- utp port はそのまま。
- linux kernel patch   1024 → 65535


##WebProxy##

[技術情報](http://bit.ly/UHbHlb)

- redis 動作nginx のポート番号を管理
- nginx ドメイン名からポート番号を引き当てる
- location をコンテナの状況で分岐処理する。


##failover.lua##

- コンテナ停止時、502が発生するので、フェイルオーバー処理に移る
- ダウンを検知したので、ダウンコンテナとして把握しておく


##dynamic-proxy.lua##

- ダウンしたコンテナには接続を振らない


##SSH Router##

[技術情報](http://github.com/mizzy/openssh-script-auth)

- git  -> Git Server
- sftp -> File Server
- ssh  -> Container

このルーティングを実現する仕組み。

- AuthorizedKeysScript
- 適切にサーバー名、パス情報を修正して実行
- 鍵情報の実態はMySQLで管理

##Deploy Servers##

[技術情報](http://bit.ly/NBbj9F)



