# ruby-notes

## 1. `method_missing` `respond_to_missing?` `const_missing`

```ruby
class MyObjectSystem < BasicObject
  DELEGATE = [:puts, :p]

  def method_missing(name, *args, &block)
    super unless DELEGATE.include? name
    ::Kernel.send(name, *args, &block)
  end

  def respond_to_missing?(name, include_private = false)
    DELEGATE.include?(name) or super
  end

  def self.const_missing(name)
    ::Object.const_get(name)
  end
end
```

## 2. `===` 与 Proc

case语句使用`===`来判定分支是否匹配，而Ruby的Proc对象正好定义`===`（call的别名）。

```ruby
even = ->(x) { (x % 2) == 0 }

p even === 4
p even === 9

num = gets.chomp.to_i
case num
when even
  puts "#{num} is even"
else
  puts "#{num} is not even"
end
```

