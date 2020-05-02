---
title: Go Install
date: 2020-05-02 14:50:44
updated: 2020-05-02 14:50:44
tags:
- Go
categories:
---


## 快速安装：
> yum install go

源码安装最新版

> git clone https://github.com/golang/go.gitcd go/git checkout 1.13版本cd srcall.bash

成功后就将go/bin/go 添加到环境变量即可。
环境变量

> goexport GOPATH=$HOME/www/go:$HOME/www/baxian/baxian_sea/doc/data/tokumxexport GOBIN=$HOME/www/bin


<!-- more -->

并在GOPATH下创建如下目录:

```bash
yangyu@bw-dev-yangyu-01[go] $tree .
.
├── bin
├── pkg
└── src

cd $GOPATH/src
go get github.com/nsf/gocode
cd github.com/nsf/gocode/
go build
go install

```


## 国内镜像
1. 使用go1.11以上版本并开启go module机制
2. 导出GOPROXY环境变量

```bash
export GO111MODULE=onexport GOPROXY=https://goproxy.cn或者export GOPROXY=https://mirrors.aliyun.com/goproxy/vim8:

cd /yangyu/.vim/pack/plugins/startgit clone https://github.com/fatih/vim-go.gitvi main.go:GoInstallBinaries

代码提示：
cd /yangyu/.vim/pack/plugins/startgit clone https://github.com/fatih/vim-go.gitvi main.go:GoInstallBinaries
go工具
go get golang.org/x/tools/cmd/godoc

```



submit text3 插件

```bash

Read the extension example and make sure you understand its contents. File an issue if you have any concerns.
Uninstall GoSublime
Run the command git clone https://margo.sh/GoSublime to install GoSublime from the development branch.
Restart Sublime Text. The status bar should instruct you to press ctrl+.,ctrl+x or cmd+.,cmd+x which creates the margo extension file. Please read all comments in the file.
If you've already switched to the new margo implementation, it might help to check the extension example for changes as there are some API breakages.



"gscomplete_enabled": true,
 // Whether or not gsfmt is enabled
"fmt_enabled": true,

```




vs code 插件安装

> https://www.cnblogs.com/yangxiaoyi/p/9692369.html

```bash

vscode setting.json
"go.formatTool": "goimports",
"go.goroot": "/usr/local/Cellar/go/1.13.5/libexec",
"go.gopath": "/Users/change/go",
"go.useLanguageServer": true,
"go.autocompleteUnimportedPackages": true,
"go.docsTool": "gogetdoc",

```

镜像，是否使用1.11以上mod模式

> go env -w GO111MODULE=on

> go env -w GOPROXY=https://goproxy.cn,direct

```bash
GOROOT：就是go的安装环境
GOPATH：作为编译后二进制的存放目的地和import包时的搜索路径。其实说通俗点就是你的go项目工作目录。通常情况下GOPATH包含三个目录：bin、pkg、src。
src目录下主要存放go的源文件
pkg目录存放编译好的库文件，主要是*.a文件;
bin目录主要存放可执行文件

go run 运行当个.go文件
go install 在编译源代码之后还安装到指定的目录
go build 加上可编译的go源文件可以得到一个可执行文件
go get = git clone + go install 从指定源上面下载或者更新指定的代码和依赖，并对他们进行编译和安装

```



