---
layout: post
title: "忘れがちで便利なRubyまとめ"
date: 2017-05-07 08:39:47
description: ""
tags:
- ruby
image: "/assets/img/por-que-usar-svg/coloridos.jpg"
categories:
- "Some series !"
twitter_text: ""
introduction: ""
---

## `**`で引数でHashを受け取る

```rb
def my_method(a, *b, **c)
  return a, b, c
end

my_method(1)
# => [1, [], {}]

my_method(1, 2, 3, 4)
# => [1, [2, 3, 4], {}]

my_method(1, 2, 3, 4, a: 1, b: 2)
# => [1, [2, 3, 4], {:a=>1, :b=>2}]
```

## Single ObjectとArrayを同じように扱う

```rb
stuff = 1
stuff_arr = [1, 2, 3]

[stuff].map { |s| s + 1 }    
# = Array(stuff).map { |s| s + 1 }

[*stuff_arr].map { |s| s + 1 }    
# = stuff_arr.map { |s| s + 1 }    
# = Array(stuff_arr).map { |s| s + 1 }
```

## ||= (@totalが空だったら実行する)
1回目は時間かかるけど2回目は時間がかからない

```rb
require 'benchmark'

def total(val)
  @total ||= (1..val).to_a.inject(:+)
end

result = Benchmark.realtime do
    total(100000000)
end
puts "1: #{result.round(2)}s"

result = Benchmark.realtime do
    total(100000000)
end
puts "2: #{result.round(2)}s"
```

## アルファベット、数字の配列を作る

```rb
('a'..'z').to_a # [*'a'..'z']
# => ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"]

(1..10).to_a # [*1..10]
# => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

## Tap
Tapを使用した方が可読性が増すかと。

```rb
class User
  attr_accessor :a, :b, :c
end

def my_method
  o = User.new
  o.a = 1
  o.b = 2
  o.c = 3
  o
end

# より読みやすくする
def my_method_readble
  User.new.tap do |o|
    o.a = 1
    o.b = 2
    o.c = 3
  end
end
```

## オブジェクトをコピーする

```ruby
base = [1, 2, 3]
copy = base
copy.reverse!
p "--- = ---"
p copy
p base
p copy.equal?(base)

base = [1, 2, 3]
copy = base.dup
copy.reverse!
p "--- dup ---"
p copy
p base
p copy.equal?(base)
```

## `def xxx(&yyy)`ブロック引数
`yield`はブロックを呼び出すもの

```ruby
def block
  yield
end

block do
  p "This is block!"
end
# "This is block!"
```
`Proc`はブロックをオブジェクト化したもの、callで呼び出すことが出来る  
Proc.newとlambdaはほぼ同義

1. 引数の取り扱い(数があっていない場合)  
Proc: 実行される  
lambda: ArgumentError

2. return, break したときの挙動  
Proc: callしたコンテキストでreturn(トップレベルで call した場合、LocalJumpError)  
lambda: ブロック内の処理を抜ける

```ruby
proc = Proc.new do
  p "This is proc!"
end
proc.call
# "This is proc!"

lambda = lambda {p "This is lambda!"}
# lambda = -> {p "This is lambda!"}
lambda.call
# "This is lambda!"
```
#### ブロック引数
&でブロック引数を受け取ることを明示
&を付けることで引数が渡ってきた時にProcオブジェクトに変換

```ruby
def test1(&block)
    yield
end
test1 do
  p "This is block1!"
end
# "This is block1!"

def test2
    yield
end
test2 do
  p "This is block2!"
end
# "This is block2!"
```