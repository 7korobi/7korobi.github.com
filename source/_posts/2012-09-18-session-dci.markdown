---
layout: post
title: "rubykaigi2012 Ruby on Rails The Bad Parts"
date: 2012-09-18 12:00
comments: true
categories: rails framework
---

[札幌rubykaigi2012](https://github.com/sprk2012/sprk2012-cfp)

[動画](http://www.ustream.tv/recorded/25420080)

railsの理想のひとつにclean code（整頓されたプログラム）があります。
けど現実のプロダクトでは難しいよね。

今のRailsは、足りない！

###Rails問題点 ⇨ 解決案###

- Helperが大きすぎる ⇨ Decolator
- partialが多すぎる  ⇨ cells
- Modelが太り過ぎる  ⇨ ライブラリ化、Acivity Layer

こうした整理を行った手法を、Data-Context-Interaction(DCI)と呼ぶ。

###Helperは我々を助けてくれない###

- viewは、それぞれ独立でいるのが理想
- 共通のHelperとか…
- Decolator を使おう！
  - [Draper](https://github.com/jcasimir/draper)
    - メソッドが変わってしまう
    - associated objects に対しても自動的にいける
  - [ActiveDecorator](https://github.com/amatsuda/active_decorator)

###partialはpartsではない###

[cells](https://github.com/apotonick/cells)

- railsにview componentを提供するもの
- mini controller、partial view で作る。

app/cells/
  sidebar_cell.rb
  sidebar/recent_tags.html.haml

= render_cell :sidebar, :recent_tags
= rencer_cell :answers, :index, @questions

###modelになるには太り過ぎ###

- とあるモデルが500ライン以上！
- アプリケーションのために小さなライブラリを作ろう
- 例：taggable とか

####Activity Layer####

ユーザーアクティビティの単位でクラス化して、module includeする。

- 複数のモデルが絡むロジックをどこに置くか？
- class My::AnswerAcceptingsController#update
- class AcceptAnswerActivity#do_process

Common Point of Common Pitfalls
アプリケーションロジックの適切な置き場所を、railsが提供していないという問題。




