---
layout: post
title: "MongoDB Update 治った"
date: 2011-12-07 20:12
comments: true
categories: MongoDB fail
---

治りました。
MongoDBでは、データ構造がおかしい場合、アップデートを受け付けなくなることがあるようです。

状況のおさらい
- update.$push しても要素が変化しない。
``` ruby
MONGODB giji['chr_sets'].find({:_id=>"sf"}).limit(-1)
=> {"_id"=>"sf", "chr_jobs"=>[...10 elements...], ...}

MONGODB giji['chr_sets'].update({"_id"=>"sf"}, {"$push"=>{"chr_jobs"=>{"job"=>"test", "face_id"=>"test", "_id"=>"test"}}})
MONGODB giji['chr_sets'].update({:_id=>"sf"}, {"$push"=>{"chr_jobs"=>{"job"=>"test", "face_id"=>"test", "_id"=>"test"}}})

MONGODB giji['chr_sets'].find({:_id=>"sf"}).limit(-1).sort([[:chr_set_id, :asc]])
=> {"_id"=>"sf", "chr_jobs"=>[...10 elements...], ...}
```

このときのデータ構造が問題でした。こんなのだったんですが（見やすくするため、yaml形式にしています。）
``` yml
---
'':
  csid: sf_sf10
_id: sf
admin: 黒体放射のエヴェレット解釈
chr_set_id: sf
maker: 重ね合せ猫のユニタリ変換

chr_jobs:
- face_id: sf01
  job: 通信士
  _id: sf01
...(9 items)

chr_npcs:
- caption: 明後日への道標
  face_id: sf04
  npc:
  - ! 'とたたたたんっ。<br><br><b>めざましい速さで木の洞に駆け込むと、じっと潜んだ暗闇に瞳がふたつ。<br>いちど大好きな閉所に収まると、そうかんたんに出てはこないのだ。</b> '
  - ! 'ちゅー！<br><br>　ちゅー！<br><br><b>がりがり、がりがり。ケージの縁をひっかくと、うろうろ、うろうろ右へ左へ駆け回る。木の洞に目もくれず、夜中じゅう走り続けるのだった……</b> '
  _id: sf04
  csid: sf

- caption: 明後日への道標（ナユタ）
  face_id: sf10
  npc:
  - ! 'とたたたたんっ。<br><br><b>めざましい速さで木の洞に駆け込むと、じっと潜んだ暗闇に瞳がふたつ。<br>いちど大好きな閉所に収まると、そうかんたんに出てはこないのだ。</b> '
  - ! 'ちゅー！<br><br>　ちゅー！<br><br><b>がりがり、がりがり。ケージの縁をひっかくと、うろうろ、うろうろ右へ左へ駆け回る。木の洞に目もくれず、夜中じゅう走り続けるのだった……</b> '
  _id: sf10
```

これを、こうする。
``` yml
_id: sf
admin: 黒体放射のエヴェレット解釈
chr_set_id: sf
maker: 重ね合せ猫のユニタリ変換

chr_jobs:
...(後略)
```

再度、試してみると…
``` ruby
irb(main):063:0> ChrSet.find('sf').chr_jobs << ChrJob.new(job:'test', face_id:'test')
MONGODB giji['chr_sets'].find({:_id=>"sf"}).limit(-1).sort([[:chr_set_id, :asc]])
MONGODB giji['chr_sets'].update({"_id"=>"sf"}, {"$pushAll"=>{"chr_jobs"=>[{"job"=>"test", "face_id"=>"test", "_id"=>"test"}]}})

MONGODB giji['chr_sets'].find({:_id=>"sf"}).limit(-1).sort([[:chr_set_id, :asc]])
=> [...10 elements..., #<ChrJob _id: test, _type: nil, job: "test", face_id: "test">]
```

増えた！

``` ruby
irb(main):076:0> ChrSet.find('sf').chr_jobs.pop.delete
MONGODB giji['chr_sets'].find({:_id=>"sf"}).limit(-1).sort([[:chr_set_id, :asc]])
MONGODB giji['chr_sets'].update({"_id"=>"sf"}, {"$pull"=>{"chr_jobs"=>{"_id"=>"test"}}})

MONGODB giji['chr_sets'].find({:_id=>"sf"}).limit(-1).sort([[:chr_set_id, :asc]])
=> [...10 elements...]
```

消せた！

と、いう顛末のようでした。キーに空文字を含むようなドキュメントが、この現象を引き起こしていたようです。


せっせと修正…。ドキュメントを削除して、修正版を挿入しなおすドキドキ作業。
``` ruby
ChrSet.all.each {|o| o.attributes.delete(''); o.delete; ChrSet.new(o.attributes).save; p o}  
```
