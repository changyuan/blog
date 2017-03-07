---
title: Redis is configured to save RDB snapshots
date: 2017-01-25 12:08:15
updated: 2017-01-25 12:08:15
tags:
categories:
---

今天在`redis`中执行`setrange name 1 chun` 命令时报了如下错误提示：

`(error) MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error．`

大意为：（错误）`misconf redis`被配置以保存数据库快照，但`misconf redis`目前不能在硬盘上持久化。用来修改数据集合的命令不能用，请使用日志的错误详细信息。


1. 磁盘满了，快照无法写入
2. 这是由于强制停止`redis`快照，不能持久化引起的，

运行info命令查看`redis`快照的状态，如下：

`rdb_last_bgsave_status:err`


解决方案如下：

运行　`config set stop-writes-on-bgsave-error no`　命令

关闭配置项`stop-writes-on-bgsave-error`解决该问题。