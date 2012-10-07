---
layout: post
title: "rails2_remember"
date: 2012-01-26 13:23
comments: true
categories: ruby rails
---

思い出したほうがよさそうな、Rails2の機能を三つほど。

・[delegate](http://apidock.com/rails/Module/delegate)
・[memoize](http://apidock.com/rails/ActiveSupport/Memoizable/memoize)
・[rescue_from](http://apidock.com/rails/ActionDispatch/Rescue/rescue_from)

## delegate ##

``` ruby
class A
  attr_reader :b
  def initialize
    @b = OpenStruct.new(
      foo: 'b.foo',
      bar: 'b.bar',
      baz: 'b.baz'
    )
  end
  delegate :foo, :to=>:b
  delegate :bar, :to=>:b, :prefix=>true
  delegate :baz, :to=>:b, :prefix=>"target"
  delegate :quod, :to=>:@b, :prefix=>"target"
end
# ArgumentError: Can only automatically set the delegation prefix when delegating to a method.


a = A.new
a.foo        # => b.foo
a.b_bar      # => b.bar
a.target_baz # => b.baz
a.target_quod # NoMethodError: undefined method `target_quod' for ...

```

## memoize ##

``` ruby
class A
  extend ActiveSupport::Memoizable
  def foo(a,b)
    puts 'a.foo exec!'
    a + b
  end
  memoize :foo
end

a = A.new
a.foo(1,2)      # a.foo exec! => 3
a.foo(1,2)      #             => 3
a.foo(1,2,true) # a.foo exec! => 3
a.foo(2,2)      # a.foo exec! => 4
a.foo(2,2)      #             => 4
a.instance_variable_get(:@_memoized_foo) # {[1,2] => 3, [2,2] => 4}
```

## rescue_from ##

``` ruby
class A
  include ActiveSupport::Rescuable
  class FetchException < RuntimeError; end
  class StoreException < RuntimeError; end

  rescue_from FetchException do |e| @error = "取得失敗: #{e}" end
  rescue_from StoreException do |e| @error = "保存失敗: #{e}" end

  def fetch
    raise FetchException if rand(2) == 0 
  end

  def store
    raise StoreException if rand(2) == 0 
  end

  def execute
    fetch
    store
  rescue Exception => error
    rescue_with_handler(error) ? @error : raise
  end
end

class B < A
  class StoreException < RuntimeError; end
  rescue_from StoreException do |e| @error = "保存失敗: #{e}" end

  def store
    raise StoreException if rand(2) == 0 
  end  
end

a = A.new
a.execute # => たまに "取得失敗: A::FetchException"
a.execute # => たまに "保存失敗: A::FetchException"
b = B.new
b.execute # => たまに "取得失敗: A::FetchException"
b.execute # => たまに "保存失敗: B::FetchException"
```
