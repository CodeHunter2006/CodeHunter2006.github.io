---
layout: post
title: "MongoDB 复制集、分片、备份"
date: 2020-05-03 23:00:00 +0800
tags: MongoDB
---

# Replica Set(复制集)

![MongoDB](/assets/images/2020-05-03-MongoDB_Sharding_2.png)

- mongodb 复制集基本构成是 1 主 2 从的结构，主会实时、无损的向从复制
- 客户端需要安装一个驱动包，初始化时将集群的所有 ip 地址告诉驱动，进行连接
- 集群中结点是独立的，自带互相监控投票机制，如果半数以上认为某节点宕机则自动切换，宕机重联后自动恢复功能，在集群变化时会自动通知驱动以便客户端连接最新的主
- monggodb 采用 Raft 协议作为分布式一致性协议，MySQL 用的是 MGL 协议(Paxos 协议的变种，Paxos 协议是 ZooKeeper 使用的协议)
- 从库是不允许写的，默认也不允许读，设置`rs.slaveOk()`后可读
- 复制集不可以级联，只能保持 1 主 2 从的结构

## Arbiter(仲裁者)模式

mongodb 也可以用 1 主+1 从+1Arbiter 架构，类似 Redis 主从复制+哨兵模式

- Arbiter 结点只负责仲裁，可以减少从主同步到从的数据的压力
-  避免只有两个结点的情况下，无法选出主的情况
- Arbiter 可以用性能较差的主机

## 延迟结点/隐藏结点

- 延迟结点是一种特殊的从库，可以设定一个较大的延迟更新时间，用于备份恢复
- 可以设置延迟结点为隐藏结点，不参与选主，不为客户端提供访问服务
- 添加、删除、配置结点的过程整个集群无需停机，自动调整

```javascript
cfg = rs.conf(); // 将配置文件转存为一个对象以便修改
cfg.members[2].priority = 0; // 设置不参与选主，2 是在members数组的下标
cfg.members[2].hidden = true; // 设置为隐藏，不为客户端提供服务
cfg.members[2].slaveDelay = 120; // 延迟复制的秒数
rs.reconfig(cfg); // 覆盖配置
```

# Sharding(分片)

![MongoDB](/assets/images/2020-05-03-MongoDB_Sharding_1.png)

- 通过分片机制，将数据分散到不同的 mongodb 实例(shard)上存储，这样可以提高写入、读取效率，避免单实例过高的并发
- mongodb 集群基本构成：
  - **Router(mongos)**负责对数据进行路由
    - **Balancer**可以监控各个 Shard 的 Chunk 分配情况，进行"Chunck 迁移"使 Shard 负载更均衡，但是迁移本身会很耗性能
  - **Shard**真正存储数据的实例
    - **Chunk**类似 Redis 中的 Slot，被分配在不同的 Shard 中，每个 64MB
  - **Config Servers**为 Router 和 Shard 提供配置中心功能
- mongodb 集群添加新的 shard 时会执行"自动分片"策略，但是不一定适用于业务。
- 可选的分片策略：
  - **Hash**按照 hash 分片，比较均匀，常用这个策略。
  - **Range**指定每个 Shard 对应的索引范围。如果数据集中于一个范围，则会导致性能下降，使用较少。
- 指定分片策略的过程：
  1. 激活数据库分片功能`db.runCommand({ enablesharding: "dbName" })`
  2. 指定**分片键**对集合分片。分片键就是一个索引`db.name1.ensureIndex({ id: 1 })`(1 表示正序、-1 表示逆序)，`sh.shardcollection("dbName.name1",{id: 1})`。注意上面操作要在 admin 库中操作，进行设置。
  - `{id: 1}` range 分片
  - `{id: "hashed"}` hash 分片
- 关于 Balance 时机：
  - 可以设定指定时间窗口进行迁移，注意这个窗口期避开繁忙期和备份时间
- 删除分片的过程：
  1. 确认 Balancer 是否在工作，`sh.getBalancerState()`
  2. 删除指定的 Shard 节点，`db.runCommand({ removeShard: "shardName" })`，删除动作会立即触发 Balance 动作

# 备份

## mongoexport/mongoimport

将数据导出为 json 或 csv 格式

- 适用场景：

  1. 异构平台迁移，MySQL <-> MongoDB
  2. 同平台跨大版本间的迁移， MongoDB2 -> MongoDB3

- 由于 MongoDB 不存在联表查询，所以导出的基本单位是 collection

## mongodump/mongorestore

# oplog

在 replica set 中 oplog 是 local 库中的一个定容集合(capped collection), 它的默认大小是磁盘的 5%(可设定)。
oplog 对应于 MySQL 中的 binlog，记录了所有操作。

- 通常 oplog 的容量设置为足够一个全备周期内的使用即可
- `rs.printReplicationInfo()` 显示 log 容量和可覆盖的时间长度，可作为备份周期的评估
