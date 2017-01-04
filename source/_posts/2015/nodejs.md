---
title: nodejs 基础
date: 2015-11-14 15:44:47
updated: 2015-11-14 15:44:47
tags:
categories:
---

Node.js 是单进程单线程应用程序，但是通过事件和回调支持并发，所以性能非常高。Node.js 的每一个 API 都是异步的，并作为一个独立线程运行，使用异步函数调用，并处理并发。Node.js 基本上所有的事件机制都是用设计模式中观察者模式实现。Node.js 单线程类似进入一个while(true)的事件循环，直到没有事件观察者退出，每个异步事件都生成一个事件观察者，如果有事件发生就调用该回调函数。
<!-- more -->
### 安装
```bash
	#mac
    brew install nvm
	#linux
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.29.0/install.sh | bash
	# 查看远程版本
    nvm ls-remote 
	nvm install v7.1.0
	nvm use v7.1.0

	nvm list
	# 注册一个cnpm
	npm install -g cnpm --registry=https://registry.npm.taobao.org
	# 切换源
	npm config get registry
	npm config set registry https://registry.npm.taobao.org
	

```


### 阻塞代码实例
```javascript
var fs = require("fs");

var data = fs.readFileSync('input.txt');

console.log(data.toString());
console.log("程序执行结束!");
```

### 非阻塞代码实例
```javascript
var fs = require("fs");

fs.readFile('input.txt', function (err, data) {
    if (err) return console.error(err);
    console.log(data.toString());
});

console.log("程序执行结束!");
```

