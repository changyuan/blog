---
title: javascript立即执行函数
date: 2017-01-25 12:08:15
updated: 2017-01-25 12:08:15
tags:
categories:
---

### 立即执行函数

立即执行函数：函数在定义后立即被执行，有特定的书写模式。例如：

```javascript
	(function(){
		console.log(123);
	}){};
```
或者
```javascript
	(function(){
		console.log(123);
	}())
```

> 这种模式本质上就是函数表达式(命名的或者匿名的)，在创建后立即执行；立即执行函数并不是标准的叫法，是自我理解的叫法。

错误的写法
```javascript
	function(){
		console.log(123);
	}()
```
原因：在一个表达式后面加上括号()，该表达式会立即执行，但是在一个语句后面加上括号()，是完全不一样的意思，他的只是分组操作符。

要解决上述问题，非常简单，我们只需要用大括弧将代码的代码全部括住就行了，因为JavaScript里括弧()里面不能包含语句，所以在这一点上，解析器在解析function关键字的时候，会将相应的代码解析成function表达式，而不是function声明。

所以立即执行函数需要有一个（）！！！！

### 也常常应用于局部作用域或者只执行一次的的场景

```javascript
	(function() { 
	    var days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'], 
	    today = new Date(), 
	    msg = 'Today is ' + days[today.getDay()] + ', ' + today.getDate(); 
	    alert(msg); 
	} ());
```

> 如果代码没有被包裹在立即执行函数中，那么局部变量days，today和msg都将成为全局变量，初始化代码的遗留产物。

### 立即执行函数的参数

传递参数方式如下：
```javascript
	(function (global) { 
		// access the global object via `global` 
	}(this));
```

例子：

```javascript
(function(who, when) { 
    console.log("I met " + who + " on " + when); 
} ("Joe Black", new Date()));
```

> 通常，全局变量被作为一个参数传递给立即执行参数，这样它在函数内部不使用window也可以被访问到：这种方式可以让代码在环境(除了浏览器)中更加通用：



### 立即执行函数的返回值

例子：
```javascript
var result = (function () { 
    return 2 + 2; 
}()); 


```
例子2：
先被计算并被储存在立即执行函数的闭包中
```javascript
	var result = (function(){
		var res = 2 * 9;
		return function() {
			return res;
		}
	}());
```

例子3：
立即执行函数也可以用来定义对象的属性；

```javascript
	var o = { 
	    message: (function() { 
	        var who = "me", 
	        what = "call"; 
	        return what + " " + who; 
	    } ()), 
	    getMsg: function() { 
	        return this.message; 
	    } 
	}; 
	// usage 
	o.getMsg(); // "call me" 
	o.message; // "call me" 

	
	//jquery
	var m = (function(e, t) {
		var n = 1;
		toArray: function(){
			return h.call(this)
		},
		b.fn = {
			//内容
		}
	})(window);


	var counter = (function(){
	  var i = 0;

	  return {
	    get: function(){
	      return i;
	    },
	    set: function( val ){
	      i = val;
	    },
	    increment: function() {
	      return ++i;
	    }
	  };
	}());

	//counter.get(); // 0
	//counter.set( 3 );
	//counter.increment(); // 4
	//counter.increment(); // 5

	//counter.i; // undefined i并不是counter的属性
	//i; // ReferenceError: i is not defined (函数内部的是局部变量)
```

[参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions)
