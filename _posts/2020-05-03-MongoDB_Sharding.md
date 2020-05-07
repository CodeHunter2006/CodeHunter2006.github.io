---
layout: post
title: "MongoDB 分片、备份"
date: 2020-05-03 23:00:00 +0800
tags: MongoDB
---

![MongoDB](/assets/images/2020-05-03-MongoDB_Sharding_1.png)

复制集
arbiter

# 分片

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
