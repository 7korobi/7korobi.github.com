---
layout: post
title: "formtastic"
date: 2011-12-11 01:25
comments: true
categories: MongoDB rails form
---

Formtasticというプラグインはなかなか便利です。
モデルの変更にあわせて自動でフォームの項目を調節してくれる、inputsというメソッドが最高にありがたい。

…ですが、これ、ActiveRecordには対応しているんですが、MongoDBだと動かないんですね。
他の変更も含め、Gemに固めてやろうといろいろ模索していますが、とりあえず最低限なにが必要なのかを覚書き。

``` ruby
module Mongoid
  module Document
    def self.included(base)
      def base.field(name, options = {})
        super
        記録を取る
      end
      def base.referenced_in(name, options = {})
        super
        記録を取る
      end
      def base.references_many(name, options = {})
        super
        記録を取る
      end
      def base.embedded_in(name, options = {})
        super
        記録を取る
      end
      def base.embeds_many(name, options = {})
        super
        記録を取る
      end
      def base.references_and_referenced_in_many(name, options = {})
        super
        記録を取る
      end
      base.class_eval do
        def column_for_attribute(attribute_name)
          なんとかする
        end
      end
      def base.content_columns
        なんとかする
      end
      def base.reflections
        relations
      end
    end
  end
end

module Formtastic
  module Inputs
    class ArrayInput
      include Base
      include Base::Stringish

      def to_html
        @object.each do |exp|
          input_wrapping do
            label_html <<
            どうにかする
          end
      end
    end
  end
end

module Formtastic
  module Helpers
    module InputHelper
      protected
      def default_input_type(method, options = {})
        大幅に書き換え。
        ActiveRecordではありえないような、
        Array型などの入力用フォームに対応する。
      end
    end
  end
end
```

けっこう長い！

実際にはもちろん、メソッド定義の中身も必要です。
