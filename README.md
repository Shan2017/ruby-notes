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

## 3. `module_function` 和 `extend self`

```ruby
module M
  extend self

  def a; end

  private
  def b; end
end

module N
  # ËùÓÐÊµÀý·½·¨±ä³Éprivate
  module_function

  def c; end

  private
  def d; end
end

puts 'M singleton methods' + M.singleton_methods.grep(/a|b/).to_s
puts 'M instance methods' + M.instance_methods.grep(/a|b/).to_s
puts 'M private instance methods' + M.private_instance_methods.grep(/a|b/).to_s
puts
puts 'N singleton methods' + N.singleton_methods.grep(/c|d/).to_s
puts 'N instance methods' + N.instance_methods.grep(/c|d/).to_s
puts 'N private instance methods' + N.private_instance_methods.grep(/c|d/).to_s
puts

class C
  include M
end

class D
  include N
end

puts 'C methods' + C.methods.grep(/^[a|b]$/).to_s
puts 'C private methods' + C.private_methods.grep(/^[a|b]$/).to_s
puts 'C instance methods' + C.instance_methods.grep(/^[a|b]$/).to_s
puts 'C private instance methods' + C.private_instance_methods.grep(/^[a|b]$/).to_s
puts
puts 'D methods' + D.methods.grep(/^[c|d]$/).to_s
puts 'D private methods' + D.private_methods.grep(/^[c|d]$/).to_s
puts 'D instance methods' + D.instance_methods.grep(/^[c|d]$/).to_s
puts 'D private instance methods' + D.private_instance_methods.grep(/^[c|d]$/).to_s
```
