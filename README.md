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

## 4. `SimpleDelegator` 类和 `DelegatorClass` 方法和 `Forwardable` 模块

Delegator

```ruby
class C
  def test
    puts "test"
  end
end

require 'delegate'

class D < SimpleDelegator; end
D.new(C.new).test

class E < DelegateClass C; end
E.new(C.new).test
```

Forwardable

```ruby
class C
  def test
    puts "test"
  end
end

require 'forwardable'

class F
  extend Forwardable

  def initialize
    @c = C.new
  end
  def_delegator :@c, :test
end

class G
  extend SingleForwardable
  
  @c = C.new
  def_delegator :@c, :test
end

F.new.test
G.test
```

## 5. `retry` 指数退避重试

```ruby
retries = 0

begin
  # TODO: 正常代码
rescue XXError => e
  raise if retries >= 3
  retries += 1

  logger.warn "API failure: #{e} , retrying ..."

  sleep 5**retrying
  retry
end
```

## 6. `arity` `~`

```ruby
func = -> (x, y=1) { x + y }

# 简单的说，arity方法返回方法或者Proc的需要的参数
# 如果不需要参数，返回0
# 如果确定需要n个参数，返回n
# 如果有n个可选参数，返回-n-1
arity = func.arity

# 通过一元补充操作符（~）来得到有几个参数是必须的
args_size = ~ arity

p args_size
```

## 7. equal?、eql?和==的区别

equal?-内存地址相同的对象，BasicObject的实例方法。
- 该方法不应该被子类覆盖
- 比较的是两个对象在内存中是否相同，是否有同一个object_id值

eql?(Object的实例方法)和==(BasicObject的实例方法)可以在子类被改写。
Object 将 eql? 方法定义为 equal? 方法的同义词，那些重写了 eql? 方法的类通常将其作为更加严格的 == 操作符，即不允许类型转换。

        1 == 1.0    # true
        1.eql? 1.0    # false

## 8. kind_of?, is_a?, instance_of? 的区别

`obj.kind_of? (klass)  ->  true or false`
判断klass是否是obj的类，或者***超类***，或者被mixin的***模块***

`obj.is_a? (klass)  ->  true or false`
和kind_of? 一样

`obj.instance_of? (klass)  ->  true of false`
判断obj是否是由klass生成的实例

Ruby Code

    module Mother; end
    
    class Father
      include Mother
    end
    
    class Son < Father; end
    
    son = Son.new
    
    p son.kind_of? Son         # true
    p son.kind_of? Father      # true
    p son.kind_of? Mother      # true
    
    p son.is_a? Mother         # true
    
    p son.instance_of? Son     # true
    p son.instance_of? Father  # false
    p son.instance_of? Mother  # false

## 9. clone()和dup()的区别

clone 和 dup 方法都会返回一个调用它们对象的浅拷贝，即如果拷贝对象有指向其他对象的引用，那么只有这些引用被拷贝，而引用的对象本身不被拷贝。

如果拷贝对象定义了 initialize_copy （拷贝构造函数）的方法，那么 clone 和 dup 就会简单的分配一个新的实例空间（该实例所属类与被拷贝对象相同），然后在其上调用 initialize_copy 的方法。

类也可以直接重写 clone 和 dup 的方法。

clone 和 dup 的区别有两点，其一，clone 会拷贝一个对象被冻结和受污染的状态，而 dup 仅仅拷贝受污染的状态; 其二，clone 方法拷贝一个对象所有的单键方法，而 dup 不会。

ruby中的dup和clone都是shallow复制, 只针对object的第一级属性。
下面是一段API文档的clone的代码：

    class Klass
      attr_accessor :str
    end
    s1 = Klass.new      #=> #<Klass:0x401b3a38>
    s1.str = "Hello"    #=> "Hello"
    s2 = s1.dup      #=> #<Klass:0x401b3998 @str="Hello">
    s2.str[1,4] = "i"   #=> "i"
    p s1.inspect          #=> "#<Klass:0x401b3a38 @str=\"Hi\">"
    p s2.inspect          #=> "#<Klass:0x401b3998 @str=\"Hi\">"

## 10. Comparable模块和Enumerable模块

Comarable基于`<=>`的，所以需要定义`<=>`方法
`Enumerable#sort, Enumerable#max` 方法都是基于`<=>`的。另外如果你定义了`<=>`，然后再include Comparable，你将免费得到<=, <, >=, >以及between方法。 
Enumerable的类只需要实现each就可以获得所有如collect, map, sort这样的方法。

## 11. `&.`运算符

***[link](http://mitrev.net/ruby/2015/11/13/the-operator-in-ruby/)***

before
```ruby
if account.try(:owner).try(:address)
...
end
```

after
```ruby
account&.owner&.address
```

## 12. `*` and `**`

```ruby
# Splat Operator (*) 

# When calling methods

arguments = [1, 2, 3, 4]
my_method(*arguments) # any number of arguments



# or:

arguments = [2, 3, 4]
my_method(1, *arguments) # any number of trailing arguments

# or:

arguments = [1, 2]
my_method(*arguments, 3, 4) # any number of preceding arguments


# or:

arguments = [2, 3]
my_method(1, *arguments, 4) # any number of "in between" arguments

# All are equivalent to:

my_method(1, 2, 3, 4)

# Two splats (**) convert a hash into an arbitary number of keyword arguments
# This operator doesn't technically have a name

arguments = { first: 1, second: 2, third: 3 }
my_method(**arguments)

# or:

arguments = { first: 1, second: 2 }
my_method(third: 3, **arguments)

# Are equivalent to:

my_method(first:1, second:2, third:3)
```

## 13. Trigger irb as needed

```ruby
require 'irb'
IRB.start
```

## 14. Block Parameters

```ruby
class Hash
  module Using
    def using(&block)
      values = block.parameters.map do |(type, name)|
        self[name]
      end

      block.call(*values)
    end
  end

  include Using
end
```

```ruby
hash = {
  first_name: "John",
  last_name:  "Smith",
  age: 35,
  # ...
}

hash.using do |first_name, last_name|
  puts "Hello, #{first_name} #{last_name}."
end

# or even...

circle = {
  radius: 5,
  color: "blue",
  # ...
}

area = circle.using { |radius| Math::PI * radius**2 }
```

## 15. file encoding

>使用一个看似神奇实则很简单的标记规则来指定代码文件的字符编码：如果一个文件的第一行（如果第一行是 UNIX shebang #!，那么就是第二行）是注释行，Ruby 会使用 coding:\s*(\S+) 这个正则表达式来对这个注释行进行匹配，如果匹配成功那么该文件的字符编码就被设置为 $1的值。所以，可以这样将一个 Ruby 代码文件的字符编码设置为 UTF-8：


`# coding: utf-8`

因为 Ruby 只是检索字符串中是否包含 coding: 这个子字符串，所以实际上也可以这样写：

`# encoding: utf-8`

Emacs 用户可能会更喜欢这样写：

`# -*- encoding: utf-8 -*-`

另外，如果 Ruby 代码文件包含了 UTF-8 BOM，也就是说代码文件的头三个字节是 \xEF\xBB\xBF，那么 Ruby 认为这个代码文件的字符编码是 UTF-8，而不管上述的标记行： 

## 16. Hash 默认的值

默认的，当我们试图接受一个在 hash 中未定义的值，你将会得到 nil。但你可以改变这个结果在进行初始化时。 （提示：一般不要这样做除非你知道自己在干什么）

在第一个例子中，我们定义了一个默认的值为0，所以当我们 a[:a] 时会得到 0 而不是 nil：
```
a = Hash.new(0)
a[:a]
# => 0
```

我们能够传入任何形式类型到 Hash 初始函数中：
```
a = Hash.new({})
a[:a]
# => {}
```
或者是一个字符串：
```
a = Hash.new('lolcat')
a[:a]
# => "lolcat"
```

## 17. STDIN STDOUT

STDIN这一类以大写字母开头，是常量；$stdin这一类以$开头，是全局变量。
  
常量不可变，STDOUT总指向屏幕显示（除非运行ruby时在命令行设置>out 2>err之类），变量可变，所以$stdin可以替换成别的IO/File对象。
  
全局的输出方法，如print puts等，总是向$stdout输出，而非向STDOUT输出，如：
```  
  print 1 # 这时$stdout和STDOUT是一致的，输出到屏幕
  $stdout = open('output_file','w')
  print 2 # 这时输出到output_file了
  $stdout = STDOUT
  print 3 # 又输出到屏幕了 
```
同時必須注意的是，我們也用了 STDIN.gets 取代了 gets。這是因為如果有東西在 ARGV 裡，標準的gets 會認為將第一個參數當成檔案而嘗試從裡面讀東西。在要從使用者的輸入（如stdin）讀取資料的情況下我們必須明確地使用 STDIN.gets。

## 18. 强制的哈希参数

这个是在 Ruby 2.0 被引进的，你可以在定义方法时指定接受的参数为 hash 类型，像这样：

def my_method({})
end

你可以指定想要接受值的键，或者为键指定一个默认的值。下面的 a 和 b 是指定要接受值的键：
```
def my_method(a:, b:, c: 'default')
  return a, b, c
end
```
我们不给 b 传值来调用它，这时会报错：

my_method(a: 1)
`# => ArgumentError: missing keyword: b`

由于 c 有一个默认的值，我们可以仅仅只给 a 和 b 传值：

my_method(a: 1, b: 2)
`# => [1, 2, "default"]`

或者是给所有键传值：

my_method(a: 1, b: 2, c: 3)
`# => [1, 2, 3]`

我们大多时候的做法是直接传入一个 hash，这样的做法只是看起来更明显，就像下面的方式：

hash = { a: 1, b: 2, c: 3 }
my_method(hash)
`# => [1, 2, 3]`

## 19. 沙盒
    def sandbox(&proc)
      lambda do
        $SAFE = 1
        puts "in proc #{$SAFE}"
        yield
      end.call
    end

    puts $SAFE
    sandbox { puts ""}
    puts $SAFE

$SAFE只在proc中生效，proc结束之后ruby解释器会把$SAFE恢复到0。其他全局变量不行
