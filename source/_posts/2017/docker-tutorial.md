---
title: Docker入门
date: 2017-07-31 15:48:22
updated: 2017-07-31 15:48:22
tags:
categories:
---

## 使用Docker能为我们解决什么问题？

可以快速思考开发和部署没有Docker的应用程序的典型工作流程。

共享设置脚本吗，你真的会使用泊坞窗运行设置脚本，并分享图像生成。此映像包含应用程序运行应用程序所需的所有依赖项，包括Ubuntu等平台。因此，当您基于这个映像运行一个容器时，它不需要下载任何东西。这也确保了在构建映像时使用的软件版本将是随处使用的软件版本。

<!-- more -->
由于每个容器都基于相同的映像，所以为了更新，您只需更新构建脚本，构建映像，测试映像仍然按预期运行，然后使用新映像部署新容器，并切换旧容器实例。这样可以确保容器彼此一致。

一个容器会有一个单独的守护进程（Docker deamon） ，然后共享内核（the kernel）。

** 使用Docker可确保部署的应用程序实例之间的一致性，最大限度地减少错误，并通过像Kubernetes这样的工具，使管理可扩展基础架构更加轻松。 **

header 1 | 应用架构 | 部署架构
---|--- |---
模块化 | 包和组件 | 容器
关注点分离 | 单一责任原则 | 面向服务的体系结构 (SOA)

举个例子，一个典型的社交媒体被分割成几个部分：
- 一个核心的应用`python` or `nodejs` or `php` 或者其他语言
- 一个mongodb数据库已提供应用信息
- 一个搜索引擎，例如elasticsearch 来返回查询结果
- 一个nginx的web服务器去处理请求和响应

您将有四个容器，一个用于应用程序的每个分支。


## 术语（terminology）
- 镜像(images)用于创建容器的应用程序的文件系统和配置。想了解更多关于码头工人的形象，运行Docker视察高山。在演示中，你使用泊坞窗拉命令下载高山图像。当你执行命令Docker运行Hello World，它也没有一个码头工人拉幕后下载你好世界图像。
- 容器 (containers) 运行Docker容器图像实例的运行实际的应用。容器包含应用程序及其所有依赖项。它与其他容器共享内核，并作为主机操作系统上的用户空间中的一个隔离进程运行。你创造了一个你下载的`alpine image`。你可以用`docker ps`这个命令列出所有的容器。
- Docker守护进程(Docker daemon) 管理大楼的主机上运行的后台服务，运行和分配的docker containers.
- docker client 命令行工具，允许用户与后台交互和Docker Deamon.
- Docker Store  docker 商店 你可以去下载使用别人做好的容器，插件等。

## installion

`uname -r` 查看版本且必须大于3.1才能使用docker

### ubuntu

```bash
wget -qO- https://get.docker.com/ | sh
sudo usermod -aG docker runoob  // 非root用户执行
sudo service docker start
//执行这个想去找hello-world 镜像，然后运行
docker run hello-world
```


### centos

```bash
yum -y install docker
service docker start
docker run hello-world


## 使用或者脚本安装
sudo yum update
curl -fsSL https://get.docker.com/ | sh
sudo service docker start
sudo docker run hello-world

```




## Usage


```bash
sudo docker version
> - Client:  
> -  Version:      1.13.1
> -  API version:  1.26
> -  Go version:   go1.7.5
> -  Git commit:   092cba3
> -  Built:        Wed Feb  8 06:50:14 2017
> -  OS/Arch:      linux/amd64
> - 
> - Server:  
> -  Version:      1.13.1
> -  API version:  1.26 (minimum version 1.12)
> -  Go version:   go1.7.5
> -  Git commit:   092cba3
> -  Built:        Wed Feb  8 06:50:14 2017
> -  OS/Arch:      linux/amd64
> -  Experimental: false


sudo docker pull ubuntu
sudo docker images

sudo docker ps -a  


//Docker容器连接
docker run -d -P training/webapp python app.py
docker run -d -p 5000:5000 training/webapp python app.py
docker run -d -p 127.0.0.1:5001:5002 training/webapp python app.py
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
//自定义
docker run -d -P --name my_container_name training/webapp python app.py

docker run --name static-site -e AUTHOR="Your Name" -d -P dockersamples/static-site

docker ps
docker ps -l
docker ps -a

> -d: 后台运行
> -P: 创建
> -p: 端口或者地址
>- e: 是如何将环境变量传递给容器
>- name:  自定义容器名称

```


## webapps whit docker


### Run a static website in a container

在[这里](https://github.com/docker/labs/tree/master/beginner/static-site) 有一个写好的容器下载
```
 docker run -d dockersamples/static-site
 docker ps // 查看
	
 docker port static-site
 //run
 docker run --name static-site-2 -e AUTHOR="Your Name" -d -p 8888:80 dockersamples/static-site
 open localhost:8888/

 //stop
 docker stop a7a0e504ca3e // 停止
 docker rm a7a0e504ca3e   // 删除

 //或者直接
 docker rm -f static-site-2


 docker ps
 docker images
```

## docker images

可以在[docker Store](https://store.docker.com/) 使用官方或者自己的镜像
```
//加载ubuntu 16.04
docker pull ubuntu:16.04  
// 拉取latest
docker pull ubuntu
```

## Other

其他可以发布自己的store，或者[部署多个app的方法](https://github.com/docker/labs/blob/master/beginner/chapters/votingapp.md)。


[教程参考](https://github.com/docker/labs/blob/master/beginner/readme.md)