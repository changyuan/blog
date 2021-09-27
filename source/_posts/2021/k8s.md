---
title: k8s
date: 2021-09-27 09:50:08
updated: 2021-09-27 09:50:08
tags:
	- Go
categories:
---


## Go Dockerfile 构建

> FROM golang:1.15

docker有一个基本镜像叫做scratch，它是一个空的镜像，在临时基础镜像上运行的应用程序只能访问内核

由于需要依赖cgo，所以我们使用scratch无法满足需求，我们需要另外一个运行时基础镜像alpine，看下dockerhub官方的介绍，它也仅仅只有5MB大小。

生成镜像，使用当前文件构建一个镜像

```bash
docker build -t changcrazy/service.doumi.com:v1  .
```

启动一个容器

``` bash
 docker run -d centos_nginx:v1 /usr/local/nginx/sbin/nginx -g "daemon off;"

```
<!-- more -->

Dockerfile文件

``` bash
# 打包依赖阶段使用golang作为基础镜像
FROM golang:1.15 as builder

# 启用go module
ENV GO111MODULE=on \
    GOPROXY=https://goproxy.cn,direct

WORKDIR /app

COPY . .

# 指定OS等，并go build
RUN GOOS=linux GOARCH=amd64 go build .

# 由于我不止依赖二进制文件，还依赖views文件夹下的html文件还有assets文件夹下的一些静态文件
# 所以我将这些文件放到了publish文件夹
RUN mkdir publish && cp toc-generator publish && \
    cp -r views publish && cp -r assets publish

# 运行阶段指定scratch作为基础镜像
FROM alpine

WORKDIR /app

# 将上一个阶段publish文件夹下的所有文件复制进来
COPY --from=builder /app/publish .

# 指定运行时环境变量
ENV GIN_MODE=release \
    PORT=80

EXPOSE 80

ENTRYPOINT ["./toc-generator"]

```

## docker go grpc

``` bash
# 构建
# docker run --name=base.domain.com -itd -p 8080:8080 -v /d/keguan/go-base-framework:/home/www/base.domain.com changcrazy/go-base:v1.0  /bin/bash
# 打包依赖阶段使用golang作为基础镜像
FROM golang:1.15.10 AS builder

# www
RUN groupadd --gid 5000 www \
  && useradd --home-dir /home/www --create-home --uid 5000 \
    --gid 5000 --shell /bin/sh --skel /dev/null www

ENV GO111MODULE=on  GOPROXY=https://goproxy.cn,direct
ENV GOROOT=/usr/local/go  PATH=$PATH:/usr/local/go/bin  GOPATH=$HOME/go  PATH=$GOPATH/bin/:$PATH

WORKDIR /home/tools

# install protoc
RUN wget http://github.com/protocolbuffers/protobuf/releases/download/v3.14.0/protobuf-all-3.14.0.tar.gz \ 
    && tar -zxvf protobuf-all-3.14.0.tar.gz && cd protobuf-3.14.0/ && ./configure && make && make install

ENV LD_LIBRARY_PATH=/usr/local/lib

WORKDIR /home/www/base.domain.com

COPY go.mod .
COPY go.sum .
RUN go mod download
# github 的protoc 版本的新版本
# RUN go get github.com/golang/protobuf/protoc-gen-go
# github 的protoc 版本的旧版本,如果使用etcd集群需要使用老版本
# RUN go get github.com/golang/protobuf/protoc-gen-go@v1.3.2

# google 的protoc 版本的 新版本
RUN go get google.golang.org/protobuf/cmd/protoc-gen-go && go get google.golang.org/grpc/cmd/protoc-gen-go-grpc

# grpc gateway v1 和 swagger 如结合 etcd集群需要使用此旧版本
# RUN go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway && \
    # && go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger

# grpc gateway v2 版本 和 swagger插件
RUN go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway \
    && go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2


COPY . .

# RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o web_serve main.go
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -installsuffix cgo -o web_serve main.go

EXPOSE 8080

ENTRYPOINT [ "./web_serve" ]

# 运行阶段指定alpine作为基础镜像

# FROM golang:1.15.10-alpine

# WORKDIR /home/www/base.domain.com

# COPY --from=build /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# COPY --from=builder /home/www/base.domain.com/web_serve .

# RUN chown www:www web_serve

# USER www:www

# ENV GIN_MODE=release

# EXPOSE 8080

# ENTRYPOINT [ "./web_serve" ]

```

##  一个简单的构建实例

```golang
package main

import (
    "net/http"
    "os"

    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    r.GET("/demo", func(c *gin.Context) {
        addr := os.Getenv("DOMAIN_ADDR")
        version := c.DefaultQuery("version", "0")
        c.String(http.StatusOK, "这是一个demo示例。addr:"+addr+", version:"+version+", k8s container version: 2.1")
    })
    r.GET("/", func(c *gin.Context) {
        c.String(http.StatusOK, "Hello World~~, 这是第一个发版的k8s集群镜像。")
    })
    r.Run(":9888") // listen and serve on 0.0.0.0:8080
}

```


对应的Dockfile

```bash
# 构建
# docker build -t changcrazy/demokaixin:v1 .
# docker run --name=demo.changkaixin.cn -itd -p 9888:9888 -v /e/p/demo.changkaixn.cn:/home/www/demo.changkaixn.cn changcrazy/kaixindemo:v1
# 打包依赖阶段使用golang作为基础镜像
FROM golang:1.17.0 AS builder

ENV GO111MODULE=on  GOPROXY=https://goproxy.cn,direct
# ENV GOROOT=/usr/local/go  PATH=$PATH:/usr/local/go/bin  GOPATH=$HOME/go  PATH=$GOPATH/bin/:$PATH

WORKDIR /home/www/demo.changkaixin.cn

COPY go.mod .
COPY go.sum .
RUN go mod download
COPY . .

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o web_serve main.go
# RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -installsuffix cgo -o web_serve main.go

# EXPOSE 9888

# ENTRYPOINT [ "./web_serve" ]


# 只要上面即可，使用下面作为运行环境文件更小
# 运行阶段指定alpine作为基础镜像

FROM golang:1.17.0-alpine

WORKDIR /home/www/demo.changkaixin.cn

COPY --from=builder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=builder /home/www/demo.changkaixin.cn/web_serve .

# RUN chown www:www web_serve

# USER www:www

ENV GIN_MODE=release

EXPOSE 9888

ENTRYPOINT [ "./web_serve" ]

```


### 推送到远程


```bash
# 构建镜像
docker build -t changcrazy/demokaixin:v1 .
# 查看构建出来的镜像
docker images
# 使用 构建出来的镜像运行容器
docker run --name=demo.changkaixin.cn -itd -p 9888:9888 -v /e/p/demo.changkaixn.cn:/home/www/demo.changkaixn.cn changcrazy/kaixindemo:v1


```
