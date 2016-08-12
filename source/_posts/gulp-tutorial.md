---
title: gulp tutorial
date: 2016-07-15 10:58:29
updated: 2016-07-15 10:58:29
tags:
categories:
---
### 安装nodejs
之前已经安装nodejs,npm 了

``` cli
  //安装中文镜像源文件
  npm install cnpm -g --registry=https://registry.npm.taobao.org
  npm uninstall xxx
  //查看已经安装的插件
  npm list
```
<!-- more -->
### 安装gulp,新建package.json文件

全局安装 `cnpm install gulp -g` ,之后  `cnpm init` 初始化, `cnpm install gulp --save-dev`
我们全局安装了gulp，项目也安装了gulp，全局安装gulp是为了执行gulp任务，这句本地安装gulp则是为了调用gulp插件的功能。
查看 `package.json` 帮助文档，命令提示符执行 `cnpm help package.json`。

### 安装gulp插件,运行demo
gulpfile.js是gulp项目的配置文件，是位于项目根目录的普通js文件（其实将gulpfile.js放入其他文件夹下亦可）,下面是一个demo：

``` javascript

  //导入工具包 require('node_modules里对应模块')
  var gulp = require('gulp'), //本地安装gulp所用到的地方
    less = require('gulp-less');

  //定义一个testLess任务（自定义任务名称）
  gulp.task('testLess', function () {
    gulp.src('src/less/index.less') //该任务针对的文件
        .pipe(less()) //该任务调用的模块
        .pipe(gulp.dest('src/css')); //将会在src/css下生成index.css
  });

  gulp.task('default',['testLess', 'elseTask']); //定义默认任务 elseTask为其他任务，该示例没有定义elseTask任务

  //gulp.task(name[, deps], fn) 定义任务  name：任务名称 deps：依赖任务名称 fn：回调函数
  //gulp.src(globs[, options]) 执行任务处理的文件  globs：处理的文件路径(字符串或者字符串数组)
  //gulp.dest(path[, options]) 处理完后文件生成路径
```
运行 `gulp testLess`、 `gulp default` or  `gulp`


### gulp 和 laravel 的使用

```
npm install -g gulp
gulp -v

```
laravel 包文件中包含 package.json 文件， `npm install` 安装包含的glup和laravel-elixir

#### Elixir 使用
包含 `gulpfile.js`文件
``` javascript
elixir(function(mix) {
    mix.less('app.less');
});
```
mix.less 任务可以用于编译Less文件，在本例中该文件名为 app.less ，这个文件位于 resources/assets/less 目录下，其内容如下


[Laravel Elixir](http://www.tuicool.com/articles/YRV7Fz)
