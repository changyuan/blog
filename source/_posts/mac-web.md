---
title: mac 常见环境安装配置
date: 2016-07-29 13:42:55
updated: 2016-07-29 13:42:55
tags:
categories:
---

## Homebrew
``` bash
  //install
  ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

  //use
  brew search /apache*/
  brew install wget
  brew uninstall wget

```
> [官网](http://brew.sh/)

<!--more-->
## zsh bash
```bash
   sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
｀cat /etc/shells｀ 查看所有的sh
`chsh -s /bin/bash`  或者 `chsh -s /bin/zsh`

[zsh](http://ohmyz.sh/)
## apache
mac自带一个版本，如果安装新的
```bash
  brew install apache
```

## nginx
自动安装依赖
``` bash
  brew install nginx

  sudo nginx
  nginx -s reload|reopen|stop|quit
  nginx -t
```
默认启用的8080端口，可以更改配置文件来设置端口。
```bash
  vim /usr/local/etc/nginx/nginx.conf
```
默认访问目录：
```bash
  /usr/local/Cellar/nginx/$version/html
```
## mysql
```bash
  brew install mysql
```
安装完成之后：
We've installed your MySQL database without a root password. To secure it run:
    `mysql_secure_installation`
To connect run:
    `mysql -uroot`
To have launchd start mysql now and restart at login:
  `brew services start mysql`
Or, if you don't want/need a background service you can just run:
  `mysql.server start`

在5.7版本之后，其自动初始化数据库，用｀mysql.erver start｀ 启动 ，`mysql -uroot` 链接使用.
在之前的需要初始化数据库。
``` bash
mysql_install_db --verbose --user='whoami' --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp
```
`mysqladmin -u root password 'password'`

## php

> [参考](https://github.com/Homebrew/homebrew-php)
安装完成之后：

To enable PHP in Apache add the following to httpd.conf and restart Apache:
    `LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so`

The php.ini file can be found in:
    `/usr/local/etc/php/5.6/php.ini`

✩✩✩✩ Extensions ✩✩✩✩

If you are having issues with custom extension compiling, ensure that
you are using the brew version, by placing /usr/local/bin before /usr/sbin in your PATH:

      PATH="/usr/local/bin:$PATH"

PHP56 Extensions will always be compiled against this PHP. Please install them
using --without-homebrew-php to enable compiling against system PHP.

✩✩✩✩ PHP CLI ✩✩✩✩

If you wish to swap the PHP you use on the command line, you should add the following to ~/.bashrc,
~/.zshrc, ~/.profile or your shell's equivalent configuration file:

      export PATH="$(brew --prefix homebrew/php/php56)/bin:$PATH"

✩✩✩✩ FPM ✩✩✩✩

加入launchctl启动控制
To launch php-fpm on startup:
    mkdir -p ~/Library/LaunchAgents
    cp /usr/local/opt/php56/homebrew.mxcl.php56.plist ~/Library/LaunchAgents/
    launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist

The control script is located at /usr/local/opt/php56/sbin/php56-fpm

OS X 10.8 and newer come with php-fpm pre-installed, to ensure you are using the brew version you need to make sure /usr/local/sbin is before /usr/sbin in your PATH:

  PATH="/usr/local/sbin:$PATH"

You may also need to edit the plist to use the correct "UserName".

Please note that the plist was called 'homebrew-php.josegonzalez.php56.plist' in old versions
of this formula.

To have launchd start homebrew/php/php56 now and restart at login:
  brew services start homebrew/php/php56
  brew services restart homebrew/php/php56



## mac 的一些工具

```bash
  cd ~
  //自己的环境变量
  curl -O https://raw.githubusercontent.com/donnemartin/dev-setup/master/.bash_profile
  // 一些颜色设置，并不好用（可选）
  # curl -O https://raw.githubusercontent.com/donnemartin/dev-setup/master/.bash_prompt
  //unix的自定义别名
  curl -O https://raw.githubusercontent.com/donnemartin/dev-setup/master/.aliases

```
一些工具替换产品
> - Terminal → [iTerm](https://www.iterm2.com/)
> - Finder → [TotalFinder](http://totalfinder.binaryage.com/) / [Path Finder](http://www.cocoatech.com/pathfinder/)
> - Spotlight → [QuickSilver](https://qsapp.com/) / [Alfred](https://www.alfredapp.com/)

[Terminal cheatsheet](https://github.com/0nn0/terminal-mac-cheatsheet)

## xcode
xcode-select --install

## rvm
```bash
  curl -sSL https://get.rvm.io | bash -s stable
  rvm install 2.3.1
  rvm use 2.3.1
```
## nvm

```bash
  brew install nvm
  nvm install 4
  nvm use 4
```

## atom
```bash
  brew cask install atom

```
