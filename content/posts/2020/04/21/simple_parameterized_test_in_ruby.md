---
title: "在 Ruby 的測試中輕鬆做到參數化測試"
date: 2020-04-21T10:10:48+08:00
tags: [ruby,unit test,parameterized test,meta programming]
categories: [engineering]
---

只是個小短篇記錄最近的反思和實際測試能力的應用。

## 上下文（Context）

最近的兩份工作都主要是在 Rails 開發，兩份工作都是高度重視測試的團隊，不過在實際的場景不太相同一樣，前一份工作主要已經引入或者開發了改善開發效率的工具，而後者是已經有一定許多既存的測試，也有一定複雜度的架構，不過許多開發工具都還在比較早期的階段，比較沒有著墨過多在開發流程的部分。

## 遇到的問題

參數化測試是撰寫測試案例中非常好用的一種技巧，通常的使用情境在於測試的對象和步驟是一致的，但是輸入和輸出是不同的。

舉例來說，我們有一個類別如下。

```ruby
class OpGreater
  attr_reader :op1, :op2

  def initialize(op1, op2)
    @op1 = op1
    @op2 = op2
  end

  def call
    op1 > op2
  end
end
```

測試案例可以如下

```ruby
# 原本的寫法
class OpGreaterTest < Minitest::Test
  def test_given_arg1_as_100_and_arg2_as_1_returns_true
    op = OpGreater.new(100, 1)
    assert_equal true, op.call
  end

  def test_given_arg1_as_1_and_arg2_as_20_returns_false
    op = OpGreater.new(1, 20)
    assert_equal false, op.call
  end
end

# 整併起來
class OpGreaterTest < Minitest::Test
  def test_op_greater
    [
      { a: 100, b: 1, expectation: true },
      { a: 1, b: 20, expectation: false }
    ].each do |args|
      op = OpGreater.new(args[:a], v[:b])
      assert_equal args[:expectation], op.call
    end
  end
end
```

後者看起來好像乾淨很多，好像也很參數化了，但是就撰寫測試來說是滿有疑慮的。主要的考量會是 1. 測試案例本身沒辦法告訴維護或使用 `OpGreataer` 的開發人員該如何使用它、2. 測試案例的名稱本身沒辦法表達測試的情境以及 3. 如果使用 `OpGreater` 改變了行為（無論有意或無意），維護的人員沒辦法迅速知道是改壞了還是測試案例該修改。

因此如果沒有使用參數化測試，比較建議囉唆一點，每一種情境寫一種專門的測試。可是，重複的事情我就不想做那麼多次啊！重複的 code 連複製貼上都懶。

## 用一點點 Ruby 的特性就可以了

之前一直覺得要導入類似的框架或函式庫才做得到參數化測試，但是後來發現根本是自己給自己挖坑，Ruby 本身就可以做到很好的動態擴充，透過 meta programming 的技巧。

啥？meta programming 太高空？

簡單說就是（你）寫程式去（讓 Ruby）產生程式來執行。

Ruby 裡面有一個很有趣的函式叫 `define_method`，透過它，我們可以在程式執行時，才把方法產生出來。用在測試的情境就是，在測試開始跑之前，才把對應的測試案例生出來，這樣我們就可以參數化測試案例，又可以懶惰啦。

結果如下

```ruby
class OpGreaterTest < Minitest::Test
  [
    { a: 100, b: 1, expectation: true },
    { a: 1, b: 20, expectation: false }
  ].each do |args|
    scenario = "test_given_arg1_as_#{args[:a]}_and_arg2_as_#{args[:b]}_returns_#{args[:expectation]}"
    define_method("test_#{scenario}") do
      op = OpGreater.new(args[:a], args[:b])
      assert_equal args[:expectation], op.call
    end
  end
end
```

這樣看起來很難讀，那就在包裝一下吧，順便讓這個功能變成通用一點。

```ruby
module ParameterizedTestHelper
  module ClassMethods
    def test_these(scenario_base, *inputs, &block)
      inputs.each do |input|
        scenario = (scenario_base % input).gsub('\s+', '_')
        define_method("test_#{scenario}".to_sym) do
          instance_exec(input, &block)
        end
      end
    end
  end

  extend ClassMethods

  def self.included(other)
    other.extend ClassMethods
  end
end

class OpGreaterTest < Minitest::Test
  include ParameterizedTestHelper

  test_these 'given args %{args} returns %{expectation}',
             { args: { op1: 100, op2: 1}, expectation: true },
             { args: { op1: 1, op2: 20}, expectation: false } do |args:, expectation:|
    op = OpGreater.new(args[:op1], args[:op2])
    assert_equal expectation, op.call
  end
end
```

## 反思

話說就為了要自己寫還是要找函式庫這件事情卡了有一個月吧，實際的通用函數當然會更複雜一點，但是其實真的動手去寫並不會比較困難的。
而且明明自己也知道 Ruby 有這些彈性和功能可用，卻還是常常過度依賴外部功能，要多多注意了。
