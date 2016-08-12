---
title: atom tutorial
date: 2016-06-27T15:58:41.000Z
updated: 2016-06-27T15:58:41.000Z
tags: null
categories: null
---

## how to use atom

[atom-flight-manual](http://flight-manual.atom.io/using-atom/sections/atom-packages/)

在编辑器中，`Command Palette` 中输入 `init script`，由于这个输入框里使用的是 `Fuzy matching`, 你输入到 s 的时候，估计 `Application: Open Your Init Script` 已经排在第一位了，当它是第一位的时候，直接回车键就可以了...... 在 `init script` 文件里拷贝粘贴以下代码：
<!-- more -->
```coffeescript
 # move cursor across the ending symbols...
 EndingSymbolRegex = /\s*[)}>\]/'";:=-]/
 atom.commands.add 'atom-text-editor', 'custom:jump-over-symbol': (event) ->
   editor = atom.workspace.getActiveTextEditor()
   cursorMoved = false
   for cursor in editor.getCursors()
     range = cursor.getCurrentWordBufferRange(wordRegex: EndingSymbolRegex)
     unless range.isEmpty()
       cursor.setBufferPosition(range.end)
       cursorMoved = true
   event.abortKeyBinding() unless cursorMoved
```

然后用 呼出 `Command Palette`，在里面输入 `keymap`，你应该能看到`Application: Open Your keymap` 已经排在第一位了，当它是第一位的时候，直接回车键 ⏎ ...... 在 Keymap 文件里拷贝粘贴以下代码：

```
"enter": "custom:jump-over-symbol"
```

现在，用呼出 Command Palette，在里面输入 reload，你应该能看到 `Window: Reload`,启动加载生效。


## atom Snippets
编写自己的 snippets 按照格式，可以把代码片段很快输入。
打开setting->install 可以添加 snippets 和 packages
例如：

```
// apm是atom提供的一个安装工具
apm install atom-yii2 //网上找的的一个yii2的snippets，还有laravel的，php的
```
可以在atom官方网站或者github中找到

[参考1]: https://atom.io/packages/snippets
[参考2]: https://scotch.io/bar-talk/best-of-atom-features-plugins-acting-like-sublime-text



## atom packages
GOOGLE 下面的词汇，会得到一些常用的包
> favorite atom packages
> most popular atom packages
> must have packages for developer
