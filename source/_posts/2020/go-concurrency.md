---
title: Go 并发
date: 2020-05-02 15:13:48
updated: 2020-05-02 15:13:48
tags:
- Go
categories:
---

# 常见的并发模型:

进程与线程（apache） C10K 问题

> - nginx,nodejs 异步非阻塞 复杂度高

> - golang ，lua，erlang 协程方式

golang 中使用goroutine 实现并发的，channels 在多个goroutine 间的数据同步和通信，select在多个channels中选择数据的读取和写入。


# 并发与并行

并发： 指同一时刻，系统通过调度，来回切换交替的运行多个任务，“看起来”是同时进行

并行： 指同一个时刻，两个任务”真正的“同时进行

> [讲解地址](https://www.youtube.com/watch?v=cN_DpYBzKso)  搬运废书问题

将复杂的任务拆分，通过goroutine去并发执行。

> Coroutine 

* 指针，& 取地址。

修改结构体的变量内容的时候，方法传入的结构体变量参数需要使用指针，也就是结构体的地址

