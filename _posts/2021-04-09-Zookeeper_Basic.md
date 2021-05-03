---
layout: post
title: "Zookeeper 分布式设计方案"
date: 2021-04-09 22:00:00 +0800
tags: Algorithm HighConcurrency Server Zookeeper
---

Zookeeper 是一个分布式网络文件系统

- Zookeeper 是主要用于解决一系列分布式协调问题：
  - 分布式锁
  - 分布式 ID
  - 分布式配置
  - 分布式服务注册发现
  - 分布式 HA(High Availability 高可用)

# 基本结构

Zookeeper 集群的基本结果是主从复制，有一个 leader 和多个 follower，总数应是奇数。

- **Leader** 用于写和读，对 Client 提供单线程的服务，类似 Redis
- **Follower** 可以用于读

- client 向 follower 发出写请求，会被转发到 leader 处理。

- **Path Node** 结点用于保持数据(持久、临时)，结点可以形成树形结构，类似文件系统
- **Session** client 连接到任意结点，就会形成一个 session 数据，断开重联可以重用 session
  - session 是由心跳维护的
  - client 和某个结点连接临时断开后，会尝试**failover**(故障切换)重联其他结点。如果超时仍然没有连上任何结点，则判定 session 失效。
- **Watch** 监控回调，可以监控特定结点(及其子结点)变更事件

# 两步提交

写入数据时的步骤：

1. client 写请求到达 leader，learder 会向所有 follower 同时发出第一阶段写提交
2. follower 收到第一阶段提交后会立刻写入内存、写入日志，然后返回 OK 给 leader
3. leader 收到超过半数(包括 leader 自己)OK 后，就向 client 返回 OK 响应，然后向所有 follower 发送第二阶段提交
4. follower 收到第二阶段提交后，入库，写入文件

由于 Zookeeper 的写入过程包括了网络通信和 IO 操作，所以性能明显低于 Redis

- 为什么第一阶段已经过半了不能算作写成功？
  因为这个写入利用的是"append log"和内存速度较快的特性，并没有真正"落库"，只能算处于一种**软状态**。
  但是基本已经能够确定最终可以成功，这时就可以给 client 响应了。

# 选举

如果集群的 leader 挂掉了，需要从 follower 中快速选出一个新的 leader。在这个过程中系统处于不可用(阻塞)状态。

- 官方说明中，这个选举过程在 200ms 以内

选举步骤：

1. leader 挂掉，follower 都意识到 leader 断连了
2. follower 向其他 follower 发送选举消息，包含了"数据 ID"和"ServerID"
3. 谁的数据 ID 较大，则优先选择；如果数据 ID 相同，则按 ServerID 排序选择。

- 每一条写入的数据都会有数据 ID，是自增的
