---
title: linux-aliases
date: 2014-12-22 13:22:37
tags:
categories:
---

#### linux别名设置

Terminal 输入 `alias` 之后可以查看已经有的别名设置

如果要设置自己的别名如下：<!-- more -->
```bash
	alias d='date' #起别名
	unalias d  #删除别名

	alias ..="cd .."
	alias ...="cd ../.."  
	alias ....="cd ../../.."
	alias .....="cd ../../../.."
	alias ~="cd ~" # `cd` is probably faster to type though
	alias -- -="cd -"

	alias d="cd ~/Documents/Dropbox"
	alias dl="cd ~/Downloads"
	alias dt="cd ~/Desktop"
	alias p="cd ~/projects"
	alias g="git"
	alias h="history"
	alias j="jobs"
	……
```


网络上很多人整理了常用的操作

#### Mac OS 操作如下：
1. cd ~
2. curl -O https://raw.githubusercontent.com/donnemartin/dev-setup/master/.bash_profile
3. curl -O https://raw.githubusercontent.com/donnemartin/dev-setup/master/.bash_prompt
4. curl -O https://raw.githubusercontent.com/donnemartin/dev-setup/master/.aliases
5. source .bash_profile
[参考文件](https://github.com/donnemartin/dev-setup)

#### 对应linux 各个版本不一样 `vim ~/.bashrc` 文件添加：

```bash
	if [ -f ~/.aliases ]; then
    	. ~/.aliases
	fi
```
1. cd ~
2. curl -O https://raw.githubusercontent.com/donnemartin/dev-setup/master/.aliases
3. source .bashrc

##### 对于ubuntu 已经包含了 `.bash_aliases` 文件

```bash
	if [ -f ~/.bash_aliases ]; then
	    . ~/.bash_aliases
	fi
```
只需要往 `.bash_aliases` 文件中写自己的别名就可以了
