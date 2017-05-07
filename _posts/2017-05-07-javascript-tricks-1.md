---
layout: post
title: "Javascript Tricks - Part1"
date: 2017-05-07 20:35:48
image: '/assets/img/'
description: 'How to use this template'
tags:
- javascript
categories:
twitter_text: 'How to use this template'
introduction: 'How to use this template'
---

# Javascript Tricks - Part1

![](http://compedio.ro/fConsole/images/javascript-logo.png)

## 空の配列を反復処理する

```js
var arr = new Array(4);
// [undefined, undefined, undefined, undefined]

arr.map((elem, index) => index);
// [undefined, undefined, undefined, undefined]

var arr1 = Array.apply(null, new Array(4));
// [undefined, undefined, undefined, undefined]

arr1.map((elem, index) => index);
// [0, 1, 2, 3, 4]
```

## 空のパラメーターをメソッドに渡す

```
function Method(a,b,c){
	console.log(a + b + c)
}
Method('val1', , 'val3');
// Uncaught SyntaxError: Unexpected token

Method('val1', null, 'val3')
// val1nullval3

Method('val1', undefined, 'val3');
// val1undefinedval3

Method(...['va1', , 'val3']);
// val1undefinedval3
```

## 配列に重複する値を入れない

```
var arr = [...new Set([1, 2, 3, 3])];
// [1, 2, 3]
Array.isArray(arr)
// true
```

#### Setオブジェクト
各値は一意でなければならない

## `!!`でbooleanに変換する

```
function Account(cash) {  
    this.cash = cash;
    this.hasMoney = !!cash;
}
var account = new Account(100.50);  
console.log(account.cash); 
// 100.50  
console.log(account.hasMoney); 
// true

var emptyAccount = new Account(0);  
console.log(emptyAccount.cash); 
// 0  
console.log(emptyAccount.hasMoney); 
// false  
```

## `+`でnumberに変換する

```
function toNumber(strNumber) {  
    return +strNumber;
}
console.log(toNumber("1234")); 
// 1234  
console.log(toNumber("ACB")); 
// NaN  
console.log(+new Date()) 
// 1461288164385  
```

## `&&`でif文を省略する

```
if (conected) {  
    login();
}

conected && login();  
```

## `||`でDefault valueを設定する

```
function User(name, age) {  
    this.name = name || "Default User";
    this.age = age || "Default age 27";
}
var user1 = new User();  
console.log(user1.name); 
// Default User  
console.log(user1.age); 
// Default age 27

var user2 = new User("Barry Allen", 25);  
console.log(user2.name); 
// Barry Allen  
console.log(user2.age); 
// 25  
```

## array.lengthをキャッシュする

```
var array = new Array(5000)
var startTime = new Date()
for (var i = 0, length = array.length; i < length; i++) {  
    console.log(array[i]);
}
var endTime = new Date();
console.log(endTime - startTime + "ms");
```

## HTML要素を取得する

```
if ('querySelector' in document) {  
    document.querySelector("#id");
} else {
    document.getElementById("id");
}
```
querySelector(".className")は最初の要素の値を返す

## 配列の最後を取得する

```
var array = [1, 2, 3, 4, 5, 6];  
console.log(array.slice(-1)); 
// [6]  
console.log(array.slice(-2)); 
// [5,6]  
console.log(array.slice(-3)); 
// [4,5,6]  
```

## 配列を切捨てる

```
var array = [1, 2, 3, 4, 5, 6];  
console.log(array.length); 
// 6  
array.length = 3;  
console.log(array.length); 
// 3  
console.log(array); 
// [1,2,3]  
```

## ヒットしたものを全て入れ替える

```
var string = "john john";  
console.log(string.replace(/hn/g, "Replaced")); 
// "joReplaced joReplaced"  
```

## 配列をマージする

```
var array1 = [1, 2, 3];  
var array2 = [4, 5, 6];  
console.log(array1.concat(array2)); 
// [1,2,3,4,5,6]; 
console.log(array2.concat(array1)); 
// [4, 5, 6, 1, 2, 3]
```

## NodeListをArrayに変換する

```
var elements = document.querySelectorAll("p"); 
Array.isArray(elements)
// false

var after1 = [].slice.call(elements); 
Array.isArray(after1)
// true

var after2 = Array.from(elements); 
Array.isArray(after2)
// true
```

#### NodeList Object
DOMノードの集合を表すオブジェクトであり、 Node.childNodes や document.querySelectorAll メソッドの戻り値として用いられる

#### Array.from()
配列型オブジェクトや反復可能 (iterable) オブジェクトから新しい Array インスタンスを生成

## 配列をシャッフルする

```
var list = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".split("");

function shuffle(array){
	console.log(array.sort(function() {  
    	return Math.random() - 0.5 
	}))
}

shuffle(list)

// 0 <= Math.random() < 1
// Math.random()が0.5より小さい場合配列の前に来る
// Math.random()が0.5より大きい場合配列の後ろに来る
```

### Array.sort()
1. 0未満の場合、a を b より小さい添字にソート
2. 0を返す場合、a と b は互いに関して変えることなく、他のすべての要素に関してソート
3. 0より大きい場合、b を a より小さい添字にソート

## 配列の中に値があるかどうか確かめる

```
var months = ['January', 'March', 'July'];  
months.indexOf('March') !== -1; 
// true  
months.includes('July');
// true
```

## Defaul parameterを設定する

```
function myFunc1(opt){
	opt = opt != undefined ? opt : 'Default Value'
	console.log(opt)
}

function myFunc2(opt = 'default value'){
	console.log(opt)
}

myFunc1(100)
// 100
myFunc1()
// Default Value
myFunc2(100)
// 100
myFunc2()
// default value
```

## Object.prototype.constructor

```js
var string = "test"
console.log(string.constructor) 
// [Function: String]
console.log(string.constructor === String) 
// true

var num = 12345
console.log(num.constructor)
// [Function: Number]
console.log(num.constructor === Number)
// true
```
