---
layout: post
title: "忘れがちで便利なJavascriptまとめ"
date: 2017-05-07 20:35:48
image: '/assets/img/'
description: ''
tags:
- javascript
categories:
twitter_text: ''
introduction: ''
---

## 空の配列を反復処理する
`var array`は長さが3の空の配列であり、Arrayメソッドで反復処理ができない。
`var array1`はundefinedに設定された値を3つ持つ配列。値を持つので反復処理可能。
書き方を変えるならば`Array(undefined, undefined, undefined)`これと同じ。

```js
var array = new Array(3);
// [undefined, undefined, undefined]

array.map((elem, index) => index);
// [undefined, undefined, undefined]

var array1 = Array.apply(null, new Array(3));
// [undefined, undefined, undefined]

array1.map((elem, index) => index);
// [0, 1, 2]
```

## 空のパラメーターをメソッドに渡す方法3つ
`null`, `undefined`, Restパラメータを使用する。

```js
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

## 値が重複しない配列を作る方法
Setオブジェクトを使う。Setオブジェクトは各値が一意でなければならない。

```js
var arr = [...new Set([1, 2, 3, 3])];
// [1, 2, 3]
Array.isArray(arr)
// true
```

## `!!`でbooleanに変換する
`!!(変数名)`で変数に値が入っている時は`true`を返す。
値が入っていない時は`false`を返す。

```js
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

```js
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

```js
if (conected) {  
    login();
}

conected && login();  
```

## `||`でDefault valueを設定する
変数の値の指定がない場合のみ、Default valueが設定されるようになる。

```js
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
`for`で繰り返し処理を行う度に`array.length`をしないようにする。

```js
var array = new Array(5000)
var startTime = new Date()
for (var i = 0, length = array.length; i < length; i++) {  
    console.log(array[i]);
}
var endTime = new Date();
console.log(endTime - startTime + "ms");
```

## HTML要素を取得する

```js
if ('querySelector' in document) {  
    document.querySelector("#id");
} else {
    document.getElementById("id");
}
```
ちなみに`querySelector(".className")`は最初の要素の値を返すので、
複数の要素を取得したい場合は`getElementsByClassName`を使用する。


## 配列の最後を取得する

```js
var array = [1, 2, 3, 4, 5, 6];  
console.log(array.slice(-1)); 
// [6]  
console.log(array.slice(-2)); 
// [5,6]  
console.log(array.slice(-3)); 
// [4,5,6]  
```

## 配列を切捨てる
`array.length`の値を必要な長さに調整する。

```js
var array = [1, 2, 3, 4, 5, 6];  
console.log(array.length); 
// 6  
array.length = 3;  
console.log(array); 
// [1,2,3]  
```

## ヒットしたものを全て入れ替える

```js
var string = "john john";  
console.log(string.replace(/hn/g, "Replaced")); 
// "joReplaced joReplaced"  
```

## 配列をマージする

```js
var array1 = [1, 2, 3];  
var array2 = [4, 5, 6];  
console.log(array1.concat(array2)); 
// [1, 2, 3, 4, 5, 6]; 
console.log(array2.concat(array1)); 
// [4, 5, 6, 1, 2, 3]
```

## NodeListをArrayに変換する
変換するとArrayのメソッドを使えるようになる。

#### NodeList Object
DOMノードの集合を表すオブジェクトであり、`Node.childNodes`や`document.querySelectorAll`の戻り値として用いられる。

#### Array.from()
配列型オブジェクトや反復可能(iterable)オブジェクトから新しいArrayインスタンスを生成

```js
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

## 配列をシャッフルする
### Array.sort()
1. 0未満の場合、a を b より小さい添字にソート
2. 0を返す場合、a と b は互いに関して変えることなく、他のすべての要素に関してソート
3. 0より大きい場合、b を a より小さい添字にソートMath.random()は0~1の値を返す。

Math.random()が0.5より小さい場合、returnされる値が0未満になるため配列の前に来る。
Math.random()が0.5より大きい場合、returnされる値が0以上になるため配列の後ろに来る。

```js
var list = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".split("");

function shuffle(array){
	console.log(array.sort(function() {  
    	return Math.random() - 0.5 
	}))
}

shuffle(list)
```

## 配列の中に特定の値があるかどうか確かめる

```js
var months = ['January', 'March', 'July'];  
months.indexOf('March') !== -1; 
// true  
months.includes('July');
// true
```