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
    
    (use each inner implement)
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

def foo(must1, must2, ..., *args)   # or must1, *args, mustLast
    args
end
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
Class method: A::m(), A.m()
Instance method: a.m()  mark: A#m

def method (param1, param2..., paramN = 'ok')
    doing sth
    return (option, auto return last expression value)
end

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

## Variable

```
local    variable start with letter or _
global   variable start with $
instance variable start with @
class    variable start with @@
```

## Assignment

```
a, b, *c = 1, 2, 3, 4, 5
a => 1, b => 2, c => [3, 4, 5]

a, *b, c = 1, 2, 3, 4, 5
a => 1, b => [2, 3, 4], c => 5
```
