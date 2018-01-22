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

if condition then(option)
    doing sth
end

loop
    while condition do(option)
        doing sth
    end

    100.times do 
        doing sth
    end
    
    arr(hash).each do | variable1, variable2... |
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
