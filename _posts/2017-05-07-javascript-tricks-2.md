---
layout: post
title: "Javascript Tricks - Part2"
date: 2017-05-07 21:31:05
description: "忘れがちで便利なJavascriptまとめ"
tags:
- javascript
twitter_text: "Favicons, touch icons e tile icons..."
introduction: "忘れがちで便利なJavascriptまとめ"
---

# Javascript Tricks - Part2
![](http://compedio.ro/fConsole/images/javascript-logo.png)

## Function.prototype.apply()
[Function.prototype.apply()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

### 概要
指定したthisと引数(配列風のオブジェクト)で関数を呼び出す
新たなオブジェクトのためにそのメソッドを書き直すこと無く継承できる

### 配列風のオブジェクト
lengthプロパティを持つ
0以上length未満の範囲の整数を名称としたプロパティを持つ

### 呼び出し方

```js
fun.apply(this, array)
fun.apply(this, arguments)
fun.apply(this, ['eat', 'bananas'])
fun.apply(this, new Array('eat', 'bananas'))
```
### 例
#### applyを利用したコンストラクタチェーン
引数のリストの代わりに配列風オブジェクトをコンストラクタと共に利用できる

```js
// 例1
function Parent(name){
    this.name = name
}
Parent.prototype.getName = function(){
    console.log(this.name)
}
function Child(name) {
  Parent.apply(this, arguments);
}

Child.prototype = Object.create(Parent.prototype)
// Child.prototype = new Parent()
// Child.prototype = Parent.prototype
Child.prototype.constructor = Child

var P = new Parent("test")
var C = new Child("iii")
C.getName()

// 例2
Function.prototype.construct = function(args) {
  // thisはMyConstructorオブジェクト
  var Constructor = this
  var NewConstr = function() { 
    // thisはNewConstrオブジェクト
    Constructor.apply(this, args); 
  };
  NewConstr.prototype = Constructor.prototype;
  return new NewConstr();
};

function MyConstructor() {
  for (var i = 0; i < arguments.length; i++) {
    this['property' + i] = arguments[i];
  }
}

var myInstance = MyConstructor.construct(myArray);
myInstance
// NewConstr {property0: "a", property1: "b", property2: "c"}
```

#### applyをビルトイン関数と共に利用する
配列とループを駆使すること無く処理が可能

```js
var numbers = [5, 6, 2, 3, 7];
var max = Math.max.apply(null, numbers); 
// 7
var min = Math.min.apply(null, numbers);
// 2
```

## Function.prototype.bind()
[Function.prototype.bind()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

### 概要
新たな関数を生成

### 例
#### 束縛された関数を生成する

#### 部分的に適用された関数

#### setTimeout とともに

#### ショートカットを作成する


## Function.prototype.call()
[Function.prototype.call()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

### 例
#### コンストラクタを連鎖させるためにcallメソッドを使用する

#### 無名関数の呼び出しにcallメソッドを使用する

#### 関数の呼び出しや'this'を明確にするためにcallメソッドを使用する


[ビルトイン](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects)

[分割代入](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

[クロージャー](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Expression_closures)

[function*](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/function*)

[Rest parameters](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions_and_function_scope/rest_parameters)
