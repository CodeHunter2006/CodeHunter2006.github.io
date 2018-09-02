---
layout: post
title:  "Redis基本特性介绍、安装方法"
date:   2018-09-02 8:11:00 +0800
tags: Redis
---
之前用的都是SQL关系型数据库，随着网络应用并发速度的要求提高，Redis这种缓存数据库使用越来越多，有必要提前学习掌握相关知识以便实际中应用。

### Redis基本特性
* __开源__， 由VMware主持，社区活跃持续更新
* __基于内存__， 速度快适合网络和高并发
* __Key-Value数据库（NoSQL）__ ，简单快速
* __支持主从同步__, 便于进行分布式操作

### 安装方法
下载地址：[https://github.com/MSOpenTech/redis/releases](https://github.com/MSOpenTech/redis/releases)

选择合适的版本、下载后解压，就可以使用了。

### 基本使用
在命令窗口来到目录，执行"redis-server.exe redis.windows.conf"就可以启动服务器，默认是6379端口

再开一个命令窗口，执行"redis-cli.exe -h 127.0.0.1 -p 6379"可以启动服务器

执行"set key 123"

然后执行"get key"

成功读取"123"，Redis学习门已打开！