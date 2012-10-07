---
layout: post
title: "MongoDB update 失敗"
date: 2011-12-05 22:00
comments: true
categories: MongoDB fail
---

Ruby on Rails3を楽しく使っていたんだけれど・・・。

Mongoidから、embeddedオブジェクトを追加すると、いっけん動作しているように見えてデータベースに格納されない。


``` ruby
ChrSet.find('animal').chr_jobs << ChrJob.new(job:'z',face_id:'z')
MONGODB giji['chr_sets'].find({:_id=>"animal"}).limit(-1).sort([[:chr_set_id, :asc]])
MONGODB giji['chr_sets'].update({"_id"=>"animal"}, {"$pushAll"=>{"chr_jobs"=>[{"job"=>"z", "face_id"=>"z", "_id"=>"z"}]}})
=> [#<ChrJob _id: z, _type: nil, job: "z", face_id: "z">]
```
こんなふうに、MongoDB上はupdate.$pushAllという扱いをしているみたいなのだけれどなあ。


苦肉の策でオブジェクトを２つつくり、片方を削除、もう片方を更新してから保存すると、保存される。
``` ruby
MONGODB giji['chr_sets'].insert([{""=>{"csid"=>"animal"}, "admin"=>"大地の震動", "chr_jobs"=>[...], "chr_npcs"=>[...], "chr_set_id"=>"animal", "maker"=>"草原のざわめき", "_id"=>"animal"}])
```

つまり、insertはできるが、updateはできない。

そういう症状になっていることになるんだけれど・・・なんでだ？
謎が解けない

