---
layout: post
title: 闭包的一些理解
categories: [js]
description: 关于闭包的一些说明
keywords: js,closure
---

# 闭包
### 闭包的定义
>闭包：是指有权访问另一个函数作用域中变量的函数。---JavaScript高级程序设计（第三版）
>闭包：是一种特殊的对象。它由两部分构成：函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。---MDN
>个人总结：封闭了外部作用域中变量的内部函数。内部函数在定义它的外部函数之外被调用，形成了闭包。


### 典型的闭包

```js
function makeFunc() {
  var name = "Mozilla";
  function displayName() {
    alert(name);
  }
  return displayName;
}

var myFunc = makeFunc();//返回对displayName对象的引用
myFunc();
```
**说明：**display被创建时，它的作用域链（本质是指向变量对象的一个列表，只引用，但不实际包含变量对象）中包含包含对全局对象，makeFunc活动对象的引用。当display被调用时，创建一个包含arguments,name的活动对象，并将该活动对象添加到作用域链的最前端。
当makeFun函数返回后，其执行环境的作用域会被销毁，但它的活动对象仍保存在内存中，直到display函数被销毁后。

### 闭包的副作用：在循环中创建闭包
>只能取得包含函数中任何变量的最后一个值

```javascript
function createFunctions(){
	var result = new Array();
	for(var i=0;i<5;i++){
		console.log(i);//通过调用可以知道，在createFunction()被调用时执行了5次
		result[i] = function(){//result[i]的值是一个匿名函数对象，通过re[0]()调用
				return i;
		};
	}
		return result;
}

var re= createFunctions();
console.log(re);//返回包含5个匿名函数的数组
console.log(re[0]());//返回的是5
```
**说明：**创建了5个匿名函数对象，每个函数的作用域链中都保存着createFunctions函数的活动对象，所以它们引用的是同一个变量`i`。
**改进：**将`i`作为参数传入，以此固定这个变量的值
```javascript
function createFunctions(){
	var result = new Array();
	for(var i=0;i<5;i++){
		result[i] = (function (num){
			return num;
		})(i);	//立即执行函数，将i作为参数传给匿名函数，立即执行
		console.log(result[i]);
	}
		return result;
}
var re= createFunctions();
console.log(re);
```
**使用闭包的方式**
```javascript
function newFun(num){//所有的回调函数不再共享同一个环境，即拥有了不同的变量
	return function(){
		return num;
	}
}
function createFunctions(){
	var result = new Array();
	for(var i=0;i<5;i++){	
		result[i] = newFun(i);//将i作为参数传给newFun函数中的匿名函数
	}
	return result;
}
var re= createFunctions();
console.log(re[0]());//返回的是0
```

### 闭包的作用
1. 保护函数内的变量安全
2. 在内存中维持一个变量
3. 实现私有方法和函数
```java
class Person(){
	private String name;
	public String getName(){//这就相当于闭包中的闭包函数
		return name;
	}
}
```

### 闭包实践---[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures#Emulating_private_methods_with_closures)
```html
<p id="help">Helpful notes will appear here</p>
<p>E-mail: <input type="text" id="email" name="email"></p>
<p>Name: <input type="text" id="name" name="name"></p>
<p>Age: <input type="text" id="age" name="age"></p>
```

```javascript
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function makeHelpCallback(help) {
  return function() {
    showHelp(help);
  };
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    /*document.getElementById(item.id).onfocus = function() {
	      showHelp(item.help);//问题所在点
    }*/
    document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
  }
}

setupHelp();
```
**说明：**该问题的原因在于赋给 onfocus 是闭包（setupHelp）中的匿名函数而不是闭包对象；在闭包（setupHelp）中一共创建了三个匿名函数，但是它们都共享同一个环境（item）。在 onfocus 的回调被执行时，循环早已经完成，且此时 item 变量（由所有三个闭包所共享）已经指向了 helpText 列表中的最后一项。