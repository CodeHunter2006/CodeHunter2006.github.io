---
layout: post
title: "Redis分布式方案"
date: 2019-01-28 10:00:00 +0800
tags: Redis Distribute HighConcurrency
---

# 单机缓存的缺点问题

- 单个 Redis 服务器会发生单点故障(与高可用相反)，一旦主机发生问题，整个系统无法提供服务
- 在读取、写入较多的情况下，一台服务器压力过大
- 单个服务器容量有限，一般来说单台 Redis 服务器最大使用内存不要超过 20G

# 分布式带来的好处

- 高可用。某些机器宕机，仍然可以使用服务。
- 高并发。对于网络服务普遍具有"读多写少"的特点，分布式可以同时并行处理很多读的请求。高并发的一些指标：
  _ 响应时间(Response Time)。例如 Tomcat，针对先到的用户先服务，例如 500 个，后边的就会等待(没有响应)。
  _ 吞吐量(Throughout)单位时间成功传输的数据量
  _ 每秒查询率(Query Per Second，QPS)
  _ 每秒事务率(Transaction Per Second，TPS)，重点是完成的有效事务，比 QPS 小。 \* 同时并发用户数

# 分布式的扩展方式

- 垂直扩展(Scale Up)
  _ 提升单机硬件处理能力，CPU、内存、SSD 硬盘
  _ 提升单机架构性能，通过 Cache 减少 IO 次数、使用异步来增加服务吞吐率、使用无锁方式加快响应速度

垂直扩展部署简单、单机利用效率高，但是存在单点问题、造价高、维护成本高。当访问量达到一定程度时，就无法提供足够的服务性能。

- 水平扩展(Scale Out) \* 增加主机

水平扩展没有单点问题，但是通信存在各种问题、管理复杂、没有统一的时钟，存在各种需要应对的挑战。

# 主从复制（Master-Slave）模式

![Master-Slave](/assets/images/2019-01-28-Redis_distribute_1.jpg)
一个 Master 负责写、读，多个 Slave 与 Master 保持同步，负责读。Slave 可以作为别的 Slave 的 Master，形成链式结构。

对于横向扩展(一个 Master 多个 Slave)的模式，随着 Slave 数量增加、Master 的读压力会增加，无法线性增加性能，优点是距离 Master 层级少、延迟小。

对于纵向扩展（一个 Master 多层 Slave）的模式，可以随机器增加线性增加并发性能，但是缺点是层级多、延迟大。

这种主从复制模式可以降低主的读压力，但是仍然存在单点问题。

# 哨兵(Sentinel)模式

![Sentinel](/assets/images/2019-01-28-Redis_distribute_2.png)
在 Redis3.0 之前，由于 Redis 没有集群模式，所以用哨兵模式应对 Master 宕机的情况。

有一个 Master 和多个 Slave 和奇数个 Sentinel。当 Master 故障后，Sentinel 会进行投票(主观下线、客观下线)，选择一个 Slave 做为 Master 继续提供服务。

这种做的好处是没有了单点问题，缺点是无法水平扩展。

# 集群模式(Cluster)

![Cluster](/assets/images/2019-01-28-Redis_distribute_3.png)
Redis3.0 版之后，提供了 Cluster 功能，可以进行水平扩展。Redis 的分布式模式较为先进，是无中心模式，每个节点都与其他节点连接、保存数据(属于自己的一部分)、保存整个集群的状态。

### Redis 集群特点

- 所有节点彼此互联，内部以二进制协议优化传输速度和带宽
- 当集群中超过半数节点检测失败时才集群 fail
- 客户端不需要中间代理层，只需要连接任何一个可用节点即可
- Redis-Cluster 把所有物理节点映射到[0~16383]个 slot(Hash 槽)上(不一定是平均分配)，由 Cluster 负责维护
- Redis-Cluster 预分配好 16384 个哈希槽，当需要在 Redis 集群中放置一个 Key-Value 时，会先对这个 Key 使用 CRC16 计算出一个数，然后对 16384 取余计算出哈希槽位置，然后将这个 Key-Value 放置到对应的节点上

### 集群容错

- 如果某个 Master 无法访问，则开始投票，投票过程是由集群中所有 Master 参与，如果半数以上 Master 节点与某 Master 节点通信超时(cluster-node-timeout)，认为当前 Master 节点挂掉。该 Master 的某个 Slave 将被选举成为 Master
- 如果任意 Master 挂掉而没有 Slave，则集群进入 fail 状态（因为 Hash-Slot 已经不完整了）；如果集群中超过半数以上 Master 挂掉，无论是否有 Slave，集群都进入 fail 状态

### 集群节点分配

起始三个节点的话，Hash-Slot 是平均分配的。当增加第四个节点时，会从每一个节点前面拿取一部分 Slot 到新节点。

### 集群创建

Redis 官方提供了 redis-trib.rb 工具用于集群创建。至少需要 3 个 Master+3 个 Slave 共 6 个节点才能建立集群。
