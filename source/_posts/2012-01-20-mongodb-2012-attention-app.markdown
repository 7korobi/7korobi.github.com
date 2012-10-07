---
layout: post
title: "MongoDB_2012 アプリから見た弱点と対策"
date: 2012-01-20 16:05
comments: true
categories: MongoDB
---

### トランザクション的な書込操作 ###
10gen社は、今後も含めそうした方向への投資をしない。
**ameba pico**、**アニマルランド**の事例のように、トランザクションを求めない設計をしてゆくと共に、どうしても必要だが高いスケーラビリティを求めはしない、という部分については、RDBMSを併用するなどしてそれぞれの長所を活かすとよい。


### non-block Write ###
データの書き込みが完了するまで待たずにデータベースがスレッドを開放し、次の操作を受け付ける機能。
下記の点が指摘されている。

- プライマリDBへデータ送信が終わった後、Journalに入るまでのタイミングでDownした場合、情報が失われる。

- バッチwrite処理中、secondary stale. となる可能性
　　プライマリDBが大量のwriteを処理しきれず、60secぶんのOPlogを使い果たすと発生する。

[セーフモードで永続化を行う](http://d.hatena.ne.jp/babie/searchdiary?word=safely)と、これらの現象は発生しなくなる。


### chunk migration中のカウント ###
chunk migration中に[count値がずれて表示される](http://d.hatena.ne.jp/matsuou1/20110415/1302873577)現象。

（mongoDB2.0 までには対策なし。）
