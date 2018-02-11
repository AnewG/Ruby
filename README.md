# Ruby

## rvm

rvm -- ruby version manager

tips: change homebrew source [brew](https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git) & [homebrew-bottles](https://lug.ustc.edu.cn/wiki/mirrors/help/homebrew-bottles)

## RDoc & ri

If a source file is documented using RDoc, its documentation can be extracted and converted
into HTML and ri formats.

ri GC == go doc xxx
ri GC::enable (method)

[colorful format](https://stackoverflow.com/questions/24318420/colorful-format-for-titles-in-documentation-ri)

## Output

`print "xxx #{variable}"`
<br>
`puts（结尾自动加换行）`
<br>
`p（输出原本数据格式，不会转义）`

## Comments

```
=begin
xxx
=end
```
`# test comment`

## Condition

```

obj1.equal?(obj2) <=> obj1 === obj2  # 全等
obj1.eql?(obj2) => 严谨的值比较  1 != 1.0
obj1 == obj2    => 不严谨的值比较 1 == 1.0
Hash 的 key 使用的是 eql

if condition then(option)
    doing sth
end

xxx if condition

unless condition then(option)
    doing sth
end

=================

case sth
    when condition then(option)
        doing sth
    when condition, condition(or) then
        doing sth
    else
        doing sth
    end
```

## Loop

```

loop
    while condition do(option)
        if conditin 
            break
        elsif condiiton
            next(continue)
        else
            redo
        end
    end
    
    (anti-while)
    until condition do(option)
        doing sth
    end

    100.times do 
        doing sth
    end
    ---
    100.times { |i|
        doing sth
    }
    
    arr(hash).each do | variable1, variable2... |
        doing sth
    end
    
    (use each inner implement, sugar of each)
    for xx in start..end(or object)
        do(option)
            doing sth
        end
        
    (deal loop)    
    loop do
        doing sth
    end
```

## Symbol

```

sym = :foo   // or :"my name"
sym.to_s     // "foo", and to_i (int)
"foo".to_sym // :foo

lite version of string, without extra methods, like constant

puts :myvalue.object_id  #2625806
puts :myvalue.object_id  #2625806
puts "myvalue".object_id #537872172
puts "myvalue".object_id #537872152   

```

## Hash

```
person1 = { :name => "x" }
person2 = { name : "x" }
```

## Regex

```
/Ruby/i =~ "Yet Ruby xxx"
```

## Command

```
puts "#{ARGV[0]}"
```

## Files

```
file = File.open(filename)
file.each_line do |line|
    print line
end
file.close
```

## Function

```
def name
    doing sth
end

def foo(must1, must2, ..., *args)   # or must1, *args, mustLast
    args
end

def method (param1, param2..., paramN = 'ok')
    doing sth
    return (option, auto return last expression value)
end

def method(x:, y:2, z:4, **args)    # or method({"y"=>2, "z"=>4})  method("y"=>2, "z"=>4)
    doing sth
end

=================

def myloop
    while true
        yield    # execution block, injection by do..end block
    end
end
num = 1
myloop do
    puts "num is #{num}"
    break if num > 10
    num *= 2
end
```

## Library

```
require: built-in library
require_relative: current file dir relative
```

## Class

对象 | 类
---- | ---
数值 | Numeric
字串 | String
数组 | Array
散列 | Hash
正则 | Regexp
文件 | File
符号 | Symbol

```

xxx.class       ==> Array or String or other
xxx.instance_of ==> true or false
xxx.is_a        ==> check is specific class or father class or More

Class method      : A::m() A.m()
Instance method   : a.m()  mark: A#m

Class variable    : start with @@
Instance variable : start with @

module MyModule                      # C.include?(M), find order is C -> M(after) -> M(before) -> C's father
    Version = "1.0"
    
    def hello(xx)
        puts "ok #{xx}"
    end
    module_function :hello           # like js -> export
end

class HelloWorld < BaseClass         # first letter upper, default father is Object, option is BasicObject(lower level)
 
    include MyModule
 
    attr_reader :name, :xxx          # attr_writer, attr_accessor

    Version = "1.0"                  # Constant XXX::Version
    
    def initialize(myname = "Ruby")  # constructor
        @name = myname
    end
    
    public                           # below are all public method
                                     # or public :xxx=, :method
                                     # or private(internal call)
                                     # or protected(class and subclass can call by instance)
    
    # Class method
    
    class << HelloWorld
        def hello(name)
            puts "#{name}"
        end
    end
    
    class << self
        def sth(name)
            puts "#{name}"
        end
    end
    
    def HelloWorld.sth(name)
        puts "#{name}"
    end
    
    def self.sth(name)
        puts "#{name}"
    end

    # Instance method
    
    def hello
        puts "ok #{@name}"
        puts "ok #{self.name}"
        super()                      # call BaseClass hello method
        
    end
    alias father_hello hello         # or use `alias` keep father method
    undef hello                      # undef hello method, include father hello method
    
    def name                         # name getter
        @name
    end
    
    def name=(value)                 # name setter
        @name = value
    end
end
bob = HelloWorld.new("Bob")

class << bob
    def xxx                          # only in bob instance
       puts "Hi"
    end
end

# include expand class, extend expand object
module Edition
    def edition(n)
        "#{self} 第#{n} 版"
    end
end

module ClassMethods
    def cmethod
        "class"
    end
end

str = "Ruby 基础教程"
str.extend(Edition)
str.edition(5) // Ruby 基础教程 第 5 版

class TestClass
    extend ClassMethods // 类方法
    include InstanceMethods // 实例方法
end

TestClass.cmethod
TestClass.new.imethod


Redefine Operator
    class Point
        attr_accessor :x, :y
        
        def initialize(x=0, y=0)
            @x, @y = x, y
        end
        
        def +(other)
            self.class.new(x + other.x, y + other.y)
        end
        
        def -@                        # +, -, ~, ! <==>  +@, -@, ~@, !@
            self.class.new(-x, -y)
        end
        
        def [](index)
            ....
        end
    end
    point0 = Point.new(3, 6)
    point1 = Point.new(1, 8)
    point0 + point1

```

## Variable

```
local    variable start with letter or _
global   variable start with $
```

## Assignment

```
a, b, *c = 1, 2, 3, 4, 5
a => 1, b => 2, c => [3, 4, 5]

a, *b, c = 1, 2, 3, 4, 5
a => 1, b => [2, 3, 4], c => 5
```

## Operator

```
&.
    item = ary&.first // when ary not nil,call ary.first. otherwise return nil  Ruby > 2.3.0

../...
    1..5 include 5, 1...5 not include 5
    
succ
    val = "a", val.succ => "b" ....

<=>
    1  <=> 10 # -1
    10 <=> 1  # 1
    1  <=> 1  # 0
    
%w
    ary = %w(this string will become arr)
```

## Exception

```
begin            # try...catch...finally
   xxx
rescue (=> e)
   xxx           # $!(e) $@(exception locate) or e, e.class|e.message|e.backtrace
   retry         # keyword: retry
rescue Exception1, Exception2 => e3
   yyy
rescue Exception3 => e2
   zzz
ensure (option)
   xxx
end

raise
raise string
raise ExceptionClass
raise ExceptionClass, string
```

## Block

```
array = ["Ruby", "Perl", "PHP", "Python"]
sorted = array.sort { |a, b| a <=> b }      # or array.sort_by{ |item| item.attribute }
p sorted

====================

def total(from, to)
    result = 0
    from.upto(to) do |num|       # upto, pick number min to max
        if block_given?
            result += yield(num)
        else
            result += num
        end
    end
    return result
end
p total(1, 10) # 55
p total(1, 10){ |num| num ** 2 }  # 385

====================

def block_args_test
    yield()
    yield(1)
    yield(1,2,3)
end
block_args_test do |a|         # [nil], [1], [1]
    p [a]
end
block_args_test do |a, b, c|   # [nil,nil,nil], [1,nil,nil], [1,2,3]
    p [a, b, c]
end
block_args_test do |*a|        # [[]], [[1]], [[1,2,3]]
    p [a]
end
# more is nil, less is empty

====================

hello = Proc.new do |name|
    puts "Hello, #{name}."
end
hello.call("World")

modify total function:

def total2(from, to, &block)     # Proc param always last param
    result = 0
    from.upto(to) do |num|       # upto, pick number min to max
        if block
            result += block.call(num)
        else
            result += num
        end
    end
    return result
end
p total(1, 10) # 55
p total(1, 10){ |num| num ** 2 }  # 385

====================

x = y = z = 0
ary = [1, 2, 3]
ary.each do | x; y |    # x; is local variable, y is local variable(nil), z is global variable
    y = x
    z = x
    p [x, y, z]
end
p [x, y, z]
# [1, 1, 1]
# [2, 2, 2]
# [3, 3, 3]
# [0, 0, 3]

```

## Class Map

```                      
                          |--- Fixnum (couputer)
           |--- Integer --
Numeric ---               |___ Bignum (science)
           |___ Float
           |___ Rational (123.45r)
           |___ Complex  (复数, 虚数部分 123.45i)
           
OCT: 0123 or 0o123 | BIN: 0b11101 | DEC: 0d123
    
====================

Array

arr[2,4] = some_arr # replace idx 2 to 4 with a new value

交集 ary = ary1 & ary2, 并集 ary = ary1 | ary2, 差集 ary = ary1 - ary2

a = Array.new(3, [0,0,0])         # same reference
a = Array.new(3) do [0,0,0] end   # not same reference

====================

String

Here Document
<<"token" or 'token', different usage. option <<- and <<~
xxx
token

puts `cat /etc/hosts`

delete last: chop and chop! 
delete \n  : chomp and chomp!

====================

Hash

====================

Regexp

%r(pattern)
%r<pattern>

=~ | !~

至少0次非贪婪 .*?
至少1次非贪婪 .+?

====================

IO

```
