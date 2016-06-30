---
title: hexo tutorial
date: 2016-06-14T19:08:34.000Z
tags:
  - hexo
  - next
categories: 日志
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

# Quick Start

## Create a new post

```bash
$ hexo new "My New Post"
$ hexo new article
$ hexo new page tags
$ hexo new page tags
```

More info: [Writing](https://hexo.io/docs/writing.html)
<!-- more -->
## Run server

```bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

## Generate static files

```bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

## Deploy to remote sites

```bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

# 总结

## 安装

```bash
    //安装nvm,nvm是一个nodejs的版本管理器
    curl https://raw.github.com/creationix/nvm/master/install.sh | sh
    //安装nodejs 4以上的稳定版，官方有两个版本，目前是4.4.5稳定版本，这里自带的也会安装npm对应的版本
    nvm install 4

    //如果已经安装，直接安装hexo客户端
    npm install -g hexo-cli
    //有时候上面的安装命令不成功，是因为npm的源请求不到，这里安装一个国内的淘宝源
    npm install -g cnpm --registry=https://registry.npm.taobao.org
    //安装完成之后,cnpm 就可以使用了
    cnpm install -g hexo-cli
```

## 常用命令

```bash
    hexo new "postName" #新建文章
    hexo new page "pageName" #新建页面
    hexo new draft "postname" #草稿
     hexo generate #生成静态页面至public目录
    hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
    hexo deploy #将.deploy目录部署到GitHub
    hexo help  # 查看帮助
    hexo version  #查看Hexo的版本
    hexo publish [layout] postname # 从草稿中发布

    //git
    npm install hexo-deployer-git --save

    hexo deploy -g  #生成加部署
    hexo server -g  #生成加预览
```

## Layout

Layout | Path
------ | -------------
post   | source/_posts
page   | source
draft  | source/_draft

## Scaffolds(脚手架)更改默认的生成模板

> 是根据scaffolds中的模板来创建的，其中参数可以用下面的：

Placeholder | Description
----------- | -----------------
layout      | Layout
title       | Title
date        | File created date

## 重新部署

```bash
    hexo clean
    hexo generate
    hexo deploy
```

部署配置

```bash
    deploy:
    type: git
    repository: git@github.com:changyuan/changyuan.github.io.git
    branch: master
```

> 在本地生成==ssh-keygen -t rsa -C "admin@example.com"== ，然后把是生成的公钥复制到==Settings->Deploy keys中==

## 本地调试

```bash
    hexo g #生成
    hexo s #启动本地服务，进行文章预览调试
    #简化为
    hexo s -g
```

## 命令简写

```bash
    hexo n == hexo new
    hexo g == hexo generate
    hexo s == hexo server
    hexo d == hexo deploy
```

> 这个静态的web网站就被部署到了github，检查一下分支是gh-pages。gh-pages是github为了web项目特别设置的分支。

## Tag Plugins 引用块

### Block Quote

{% blockquote 作者, 作品 %} Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy. 这是引用的话。 {% endblockquote %}

--------------------------------------------------------------------------------

{% blockquote Seth Godin <http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html> Welcome to Island Marketing %} Every interaction is both precious and an opportunity to delight. {% endblockquote %}

### Code Block

{% codeblock %} alert {% endcodeblock %}

{% codeblock lang:objc %} [rectangle setX: 10 y: 10 width: 20 height: 20]; {% endcodeblock %}

{% codeblock Array.map %} array.map(callback[, thisArg]) {% endcodeblock %}

{% codeblock _.compact <http://underscorejs.org/#compact> Underscore.js %}_ .compact([0, 1, false, 2, '', 3]); => [1, 2, 3] {% endcodeblock %}

- [参考1](http://blog.fens.me/hexo-blog-github/)
- [参考2](https://hexo.io/docs/writing.html)
