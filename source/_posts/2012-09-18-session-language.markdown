---
layout: post
title: "rubykaigi2012 ruby2.0とその先"
date: 2012-09-18 12:00
comments: true
categories: rails ruby
---

[札幌rubykaigi2012](https://github.com/sprk2012/sprk2012-cfp)

#Towards Ruby 2.0: Progress of VM Internals#

[動画](http://www.ustream.tv/recorded/25419520)


##履歴と予定##

- 2012-08    Big-feature freeze
- 2012-10    Feature freeze
- 2013-02-24 Ruby 2.0 Release 20th anniversary


##Policy 開発方針##

ruby2.0の新機能をどのように実装したか。

- Compatibility
- Compatibility
- Compatibility
- Usability
- Performance


##ruby2.0でやったこと##

###Module#prepend###
- 従来、alias_method_chainでダーティ・ハックしていたことを実現するためのmethod.
- around (before-each after-each) など実現しやすい。
- prependするモジュールがサブクラス側の扱い。Object#extend はクラスの定義物より優先で処理されるが、同等の順序となる。


###Flonum(64bit)###

Floatがimmediateでないため、GCコストが莫大になる問題が発生する。
そこで、64bit向けに、IEEE754演算をimmutableオブジェクトとして提供する。
ただし、拡張ライブラリはリビルドしないと、VALUEがずれる。

- VALUE(インスタンスのメモリ上の共用体)
  - Fixnum
  - Flonum *追加*
  - Symbol
  - Qfalse
  - Qnil
  - Qtrue
  - Qundef
  - Pointer

- Flonum < IEEE754 2bit落とそう
  - IEEE754 b52-b62 with-in 768 のみFlonumと扱う。
  - rotate 3bit + 2bit mask
- Benchmarkはほぼ一定。多少遅い部分をチューンする


###backtrace API###

- caller_locations
  - lineno
  - path
  - etc...
  - 例外を沢山だすプログラムが高速化する


###set_trace_func###

- TracePoint API
  - メソッド呼び出しだけInvokeする、などの処理が可能。
- New C API


###Controllable asynchronous Interrupts###

- 終了処理の最中でも非同期割り込みは入ってきてしまうが、それでは困る処理のための機能。
- Thread.control_interrupt(TimeoutError => :on_blocking) do … end


###その他の変更###

- finish frame を取り除きました。
- allocation function を直接に変更、高速化しました。
- VM最適化：メソッド呼び出しコストが大きいので、最適化したい
- VMデータ構造訂正


###将来やりたいこと###

- C APIs for Incomplete Feature


あきらめたこと

- MVM



#Purely functional programming in Ruby#

[動画](http://www.ustream.tv/recorded/25412526)

副作用を用いないプログラミング
関数プログラミングをすると、自然に永続データ構造になる。

- 短命データ構造 変更をくわえると前のデータが失われるデータ構造
- 永続データ構造 変更前のバージョンのデータが保存されるデータ構造
  - 本来、並列動作に適している。（本セッションでの紹介内容は、スレッドセーフにしていない。）
  - テストしやすい

##関数プログラミングがしないこと##
- 変数の再代入
- オブジェクトの状態変更
- 入出力

##リスト##
Arrayは副作用を避けられないので、線形リストを実装しよう。

- while は副作用に依存する。
- Enumerable#map 内部ではpushの副作用を使う。

``` ruby
 List[1,2,3] - head: 1 tail: List[2,3]
   List[2,3] - head: 2 tail: List[3]
     List[3] - head: 3 tail: nil
      List[] - nil
```

##末尾再帰##

引数に情報をうまく蓄積するようにするテクニック。
スタックの消費を抑え、SystemStackError(!)を避けられます。
コンパイルオプションで最適化が有効になる。（Ruby2.0ではデフォルトになるかも？）

- 末尾再帰は空間効率が良い。大きい入力に有利。
- 末尾再帰でないコードは時間効率がよい。小さい入力に有利
- ただし、計算量はどちらもO(n)

``` ruby
class Cons< List
  def length
    tail.length + 1
  end
end
def Nil.length
  0
end

⇩

class List
  def length
    _length(0)
  end
end
class Cons < List
  def _length(n)
    tail._length(n + 1)
  end
end
def Nil._length(n)
  n
end
```

##Listを組み込みオブジェクトにする取り組み(#6242)##
- ※ マルチスレッド動作への対応はできていない状況です。
- 右結合の演算子 ::: で、カッコ無しで書く。
- 0 ::: S[1,2,3] => S[0,1,2,3]
- 0 ::: 1 ::: 2 ::: 3

###Lazy evaluation###
- 必要になるまで式の評価を遅延させる。
- Memoization
  - 何度実行しても一回しか評価されない。
  - 副作用がないからそれで問題ない。

###Immutable###
- map
- Queue
- Deque
- Promise
  - lazy
    - lazy(x) = delay(x.force)
    - 再帰してもスタックが溢れない
- Stream
  - 遅延リスト（必要とされるまでリストの要素がない）


###Benchmark###
- ListはArrayの7.4倍遅い！
- 繰り返しdupするようなことにおすすめ。



#Pattern Matching in Ruby#

[動画](http://www.ustream.tv/recorded/25413389)

##パターンマッチとは##

多重代入＋α

```ruby
多重代入
a, (b,c) = [1, [2,3]]
```

```ruby
パターンマッチ
[1, [2, 3] match {
case [a, [b, c]] =>
```

なぜ多重代入ではいけない？

##多重代入の問題点##
- 分解機能が弱い Arrayに限定されてしまう。
- 照合機能が弱い 個数の確認ができない。
- 代入機能がない 途中の値（[2,3]の部分）を代入できない

```ruby
Array[1,[2,3]] match
case Array[1,[b,c]] => Array[a,b,c]

List[1,[2,3]] match
case List[1,[b,c]] => List[a,b,c]
```

###分解の例###
```ruby
Email#unapply

"foo@example.com" match
case Email(user, domain) => "foo","example.com"
```


```ruby
"2012-09-15" match
case /(¥d+)-(¥d+)-(¥d+)/ =>
```

###照合の例###
```ruby
Array[1,Array[2,3]] match {
case Array[a,[b,c],d]   => nil
case Array[a,List[b,c]] => nil
case Array[0,[b,c]]     => nil
}
// MatchError
```

###代入の例###
```ruby
Array[1,Array[2,3]] match {
case Array[a,d @ Array[b,c]] => Array[a,b,c,d]
}
```

[Rubyで実際に書く方法](https://github.com/k-tsj/pattern-match)
```ruby
gem install pattern-match
```

パターンマッチは非常に強力
- ログ分析
- 正規表現のかわり
- 他にもユースケースは沢山！



###分解(Deconstructable)###
```ruby
class Regexp
  include PatternMatch::Deconstructable

  def deconstruct(val)
    m = Regexp.new
```
  with(Array.(a,b))
- Deconstructor のクラスに、Deconstructorをinclude
- Array.deconstruct(val)

照合/代入の例

Destructuring Assignment
Duck typing style

利用例
- テキスト処理（エラー処理もある）
- ツリーノード処理（HTML,XMLなど）

デザイン（文法）
- match式を新設
- case式を拡張
- メソッド引数、ブロック引数（文化的にスコシチガウ）

デザイン（分解）
- コンストラクトメソッド（コンテキストに応じた分解）
- コンストラクタ引数を適切に変える

デザイン（照合）
- 正規表現的マッチ
   *i, Fixnum, *j  が動けば…
- Mathematicaではやっている。
    Replace[{1,2,3,4,5}, {a___, b_, Shortest[c___]}]
- Setなど、正規形をもたないオブジェクトに対する照合は？





