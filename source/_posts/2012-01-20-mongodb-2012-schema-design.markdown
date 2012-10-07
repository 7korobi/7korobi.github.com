---
layout: post
title: "MongoDB_2012 スキーマデザイン"
date: 2012-01-20 16:02
comments: true
categories: MongoDB
---

## ドキュメント設計（テーブル設計） ##
[Mongoid](http://mongoid.org/docs/relations.html)を使いましょう。
embeddedオブジェクトは高速ですが、引き換えに総浚い検索がやりづらくなります。
用途の違いをわきまえて embeds / references を使い分けましょう。

また、関連データ（別ドキュメントのIDを保管するフィールド）の整合を保つため、モデルを使ってアクセスしましょう。

### 参考 ###
- [リレーショナルな関連](http://d.hatena.ne.jp/babie/20100809/1281316974)
- [スキーマ設計](http://d.hatena.ne.jp/masa_w/20101130/1291084939)
- [Mongoid 詳解](http://d.hatena.ne.jp/babie/20100809/1281316971)

ここでは、上記リンク先の内容には触れず、セミナーで紹介されたちょっとしたtipsのみお伝えします。

### 他、こまかいテクニック ###
補助的なレコードを付与して、高速でスマートなQueryingを行うための工夫について

- Tree
コメントにリプライが可能、などの、親子関係が継ぎ足されている構造です。
rootまでの祖先を辿れるよう、配列に保管しましょう。（配列データを扱える強み）
``` ruby
field :ancestors, :type => Array
# ancestors: [root_id, parent_id, self]
```

- 更新の衝突
複数のThreadが、それぞれ同一のデータをクエリして、更新操作を行ってしまうケースがあります。
こうした問題を避けるため、タイムスタンプや番号でバージョンを把握し、異なるバージョンには保存しないようにしましょう。

``` ruby
> @model = Model.find('mongo_master')
> Model.where(
    :_id => @model.id, :version => @model.version
  ).update(
    :data => 'value',  :version => @model.version + 1
  )
MONGODB db['models'].find({{"_id"=>"mongo_master"}).limit(-1)
MONGODB db['models'].update(
  {"_id"=>"mongo_master", "version"=>2},
  {"$set"=>{:data=>"value", :version=>3}}
e)
=> 194
```

また、もう一歩進めて、バージョン管理を導入するのもよいでしょう。
``` ruby
  include Mongoid::Versioning
```


## single table inheritance ##
異なるクラスの情報をひとつのテーブルに同居させることです。
こんなモデルがあったとして。

``` ruby
class Shape
class Circle < Shape
class Square < Shape
class Rect   < Shape
```

- (RDBの場合)
```
|id|type   | area| radius| length| width|
|--|-------|-----|-------|-------|------|
| 1|circle | 3.14|      1| (null)|(null)|
| 2|square |    4| (null)|      2|(null)|
| 3|rect   |   10| (null)|      5|     2|
```


- MongoDB
``` javascript
{_id:"1", type:"circle", area:3.14, radius:1}
{_id:"2", type:"square", area:4, length:2}
{_id:"3", type:"rect",   area:10, length:5, width:2}
```
