---
title: Markdown
date: 2015-03-15 00:43:20
tags:
- tools
categories:
---

## markdown 在 sublime text 3 中使用

作为Windows/Mac/Linux下强大的文本编辑器，st提供了对Markdown语言的支持。通过设置可实现markdown预览和转换功能。而本文介绍的Markdown Preview支持Mathjax语法和目录自动生成。(Windows下）,
打开st，按下组合键Control + `，出现控制台，输入
<!-- more -->

``` bash

	import  urllib.request,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();urllib.request.install_opener(urllib.request.build_opener(urllib.request.ProxyHandler()));open(os.path.join(ipp,pf),'wb').write(urllib.request.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())

```

当看到代码最后一行提示的时候说明安装成功，此时重启st，可在Preferences -> Package Settings看到Package Control。
安装markdown preview
按下键Ctrl+Shift+p调出命令面板，找到Package Control: install Pakage这一项。搜索markdown preview，点击安装。

使用
Markdown Preview较常用的功能是preview in browser和Export HTML in Sublime Text，前者可以在浏览器看到预览效果，后者可将markdown保存为html文件。

preview in browser据称是实时的，但是实践上还是需要在st保存，然后浏览器刷新才能看到新的效果，好在markdown写得多的话也不需要每敲一行看一次效果。

快捷键

新版本默认快捷键为 * ctrl+shift+m *

旧版本下面配置：
st支持自定义快捷键，markdown preview默认没有快捷键，我们可以自己为preview in browser设置快捷键。方法是在Preferences -> Key Bindings User打开的文件的中括号中添加以下代码(可在Key Bindings Default找到格式)：

 > { "keys": ["alt+m"], "command": "markdown_preview", "args": { "target": "browser"} }

"alt+m"可设置为自己喜欢的按键。

设置语法高亮和mathjax支持
在Preferences ->Package Settings->Markdown Preview->Setting Default中的第31行和36行找到

``` config

	"enable_mathjax": false,
	"enable_highlight": false,

```
将 两个false改为true即可。
语法高亮跟编辑器的主题有关，可以在Preferences ->Color Scheme找自己喜欢的主题。
关于目录生成，只要文章是按照markdown语法写作的。在需要生成目录的地方写
[TOC]
即可。

## markdown 语法参考

@changyuan

Emphasis, aka italics, with *asterisks* or _underscores_.
Strong emphasis, aka bold, with **asterisks** or __underscores__.
Combined emphasis with **asterisks and _underscores_**.
Strikethrough uses two tildes. ~~Scratch this.~~

[参考1](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
[参考2](https://guides.github.com/features/mastering-markdown/)


## emoji表情参考
:smile::thumbsup::thumbsup:
~~emo~~
[emoji表情参考](http://www.emoji-cheat-sheet.com/)
