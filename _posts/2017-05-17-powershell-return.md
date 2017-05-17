---
layout: post
title: "powershell return"
date: 2017-05-17 21:18:24
description:
tags:
- powershell
twitter_text:
introduction:
---

## 関数の`return`
* 現在のスコープを抜ける
* `return`はあってもなくてもいい
* `return`以外の他のオブジェクトも返す

```ps1
function Test-Return
{
	'Hello'
	return 12
	'Good'
}

Test-Return
# Hello
# 12
```

## クラスの`return`
* 現在のスコープを抜ける
* returnは必須(ない場合errorになる)
* `return`で指定されたものだけ返す

```ps1
class ReturnTester
{
    [int32]TestReturn()
    {
        'Hello'
        return 12
        'Good'
    }
}

$return = New-Object -TypeName ReturnTester
$return.TestReturn()
# 12
```
