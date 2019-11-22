---
layout: post
title: "Redis基本特性介绍、安装方法"
date: 2018-09-02 8:11:00 +0800
tags: Redis
---

之前用的都是 SQL 关系型数据库，随着网络应用并发速度的要求提高，Redis 这种缓存数据库使用越来越多，有必要提前学习掌握相关知识以便实际中应用。

### Redis 基本特性

- **开源**， 由 VMware 主持，社区活跃持续更新
- **基于内存**， 速度快适合网络和高并发
- **单线程**，通过单线程、异步处理方式提高并发，速度快、容易实现原子操作
- **Key-Value 数据库（NoSQL）** ，简单快速
- **支持主从同步**, 便于进行分布式操作

### 安装方法

下载地址：[https://github.com/MSOpenTech/redis/releases](https://github.com/MSOpenTech/redis/releases)

选择合适的版本、下载后解压，就可以使用了。

### 基本使用

在命令窗口来到目录，执行"redis-server.exe redis.windows.conf"就可以启动服务器，默认是 6379 端口

- redis-server 不需要用 nohup 命令启动，默认就不会输出到终端，不会随终端退出而中断

再开一个命令窗口，执行"redis-cli.exe -h 127.0.0.1 -p 6379"可以启动服务器

执行"set key 123"

然后执行"get key"

成功读取"123"，Redis 学习门已打开！
