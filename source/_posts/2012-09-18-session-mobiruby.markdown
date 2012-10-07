---
layout: post
title: "rubykaigi2012 Ruby + iOS = Super awesome!"
date: 2012-09-18 12:00
comments: true
categories: rails iOS
---

[札幌rubykaigi2012](https://github.com/sprk2012/sprk2012-cfp)


[動画](http://www.ustream.tv/recorded/25396312)

##mruby##

組み込み環境用ruby実装。

###特徴###

- 省メモリ
- 省ストレージ
- 非POSIX, only C99 → ブートローダーなどの立場で動作可能
- CRuby 拡張ライブラリと互換がない。（rubygems が使えない）

###マイルストーン###
- fiber ゲーム業界が欲しい（イベントドリブン書きづらいので）
- デバッグについて
  - エミュレーターなど開発環境でだけは、リッチな情報が欲しい。
  - mobirubyでは変数レポートなどしたい。
- コミュニケーションは、githubのみ（ML用意しない）
- 多様性（しかし）
  - JRuby Rubinius CRuby でもGemが使えるように、エクステンションの労力が少なくしようという流れの中にある。
  - extension API では、mrubyの要求を満たさない。
  - JRubyなどが extension互換性については頑張っていることと、CRubyが今後のメインストリームの位置にあればよい。
  - mruby gemsを別に作る提案もある。フレームワークを決めて、extensionを一気にコンパイルしてくれる仕組み。
  - が、今はまだ検討の段階にある。


##MobiRuby##

iPhoneアプリケーションをRubyで書く
http://mobiruby.org/

###目標###

- mruby(組み込みruby) を使ったruby開発。
- rubyの力をiOS/androidに提供する。
- 最終的にはラッピングAPIの提供まで視野に入れる。
- ダイレクトAPIの提供を残し、必要なら細かい操作が可能にする。

###来歴###

- 2012/03	きっかけ
- 2012/06	スタートアップ
- 2012/09	MAC環境α（samegame）人によっては動作しないらしい？

###現状###

- iOS と Ruby が大好き、という人専用（現状）
- 128bit 整数（C long long）サポートがまだ
- デバッグ情報を出す機能がまだ
- appleの審査を通すため、eval、define_methodなどの動的機能はない。
- ObjC に機能がないため、特異メソッド定義ができない。

###RoadMap###

- Fix BUG
- cocoa bridge 完了
- Documentation
- Packaging

###構成###

- iOS
  - mruby
    - mruby-cfunc Cブリッジ
      - mruby-common
      - mruby-cocoa Cocoaブリッジ
      - mruby-ios iOS特化機能、iOSブートストラップ。将来はラッパライブラリ

###mruby-common###
- iPhone、Android 共通機能
- POSIX、ランダムなど

###mruby-cocoa Cocoaブリッジ###
- cocoaの動的支援（文字列リクエストでクラスのポインタを取得、など）を使っている。
- UIウィンドウからインスタンスを作ってVisibleにする。など
- ObjC-Class < Ruby-Class < ObjC-Class
- ruby からの呼び出時は、リフレクションする。
  - define で定義すると、ruby、objC 両方からコール可能な定義になる
  - Class#_method で、rubyからobjCのメソッドを呼ぶ。
- ObjC object と可換な Ruby object
- 今バージョンはメモリリークがある。（リセット連打で落ちたりする）
- メモリ管理 ObjCメモリ管理を Swizzle して、実態はmrubyの管理にしている。
  - 重い！全メモリアクセスをフックするため
- マルチスレッド非対応。複数のVMを同アプリで起動し、スレッドのように扱うアイデアで検討中


###mruby アプリケーション###
- app.rb
  - hello.rb



