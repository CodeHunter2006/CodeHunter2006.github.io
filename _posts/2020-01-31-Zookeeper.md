---
layout: post
title: "Zookeeper基本使用"
date: 2020-01-31 21:00:00 +0800
tags: Server
---

![remote work](/assets/images/2020-01-31-Zookeeper_1.png)
Zookeeper 是一个开源的分布式文件系统，是 Hadoop 的组件，最早起源于雅虎研究院。可以用于网络锁、配置中心、网络队列。
因为雅虎内部网络项目特别多，需要一个协调的组件，所以就起名 Zookeeper——动物管理员。

# 客户端命令

`zkCli.sh`
在 zk 所在主机直接运行即可以客户端模式访问服务

`ls /path`
查看 path 下的节点列表

`get /path`
查看某个节点自身存储的数据

`create /path str`
创建节点并关联字符串到该节点

- `-e`创建临时节点

`set /path str`
修改节点关联的字符串

`delete /path`
删除节点，如果节点有子节点，则无法删除

`rmr /path`
递归删除所有节点
