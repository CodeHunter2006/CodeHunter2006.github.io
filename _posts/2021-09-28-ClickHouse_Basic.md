---
layout: post
title: "Click House 基础"
date: 2021-09-28 22:00:00 +0800
tags: ClickHouse
---

![ClickHouse](/assets/images/2021-09-28-ClickHouse_Basic_1.png)

- 优点：

  - (在较大数据规模下)速度快，相比 MySQL、Hadoop 快百倍
  - 支持 SQL，无需学习专用查询语法
  - 支持各种表引擎、集成引擎，提供更丰富的功能
  - 分布式架构，规模可以线性增长、可靠性强
  - 开源
  - No Hadoop ecosystem，无需依赖复杂的 Hadoop 环境

- When (not) to use

  - When to use
    - 大量/批量数据插入
    - 无(极少)的数据修改
    - 宽表(列较多)
    - 对列有较多聚合操作
    - 快速查询
  - When not to use
    - OLTP: 要求数据事务/更新
    - Key-Value：只要单行数据
    - "极度泛化"的数据：需要大量 Join 的情况
    - Blob 或 document 的存储

- 设计点(OLAP 通用设计)

  - 向量化处理
    一般操作都要对数组中保存的一批数据处理，而不是单个数据
  - 并行化
    能够并行的处理都在主机、CPU 间并行处理，尽量利用资源
  - 列式存储
    每列数据单独存储在一个文件
  - 列数据压缩
    用普通的压缩算法(LZ4、ZSTD)压缩，但是由于单列类型相同、重复多，所以压缩效果好；
    另外对特殊的类型做了优化压缩。
  - 数据块内有序
    便于快速查询
  - 数据后台重排
    不影响实时插入和查询，充分利用资源
  - 稀疏索引
    对数据的(聚集)索引用稀疏索引实现，这样可以节省容量，同时也利用了列数据批量压缩的特性。

- 分布式

  - CK 是比较粗暴的 Share-Nothing 方式，不同分区不共享数据
  - 一个集群由多个 shard 组成，每个 shard 可以有多个 replica
  - shard 内的 replica 存储的数据是一致的，在查询时可以分担流量
  - shard 内其中一个 replica 是主，其它是备

- 数据存储时的文件

  - `xxx.idx`
    存储稀疏索引，一般是每 8192 个元素一个索引
  - `xxx.mrk`
    标记文件，标记数据块压缩后，每块数据的起始、终止偏移量
  - `xxx.bin`
    真正存储数据，存储压缩后的列数据

- 缩写 CK 或 CH
- 由俄罗斯 xxx 在 xxx 开源
- OLAP
- 列存储，适合存储宽表数据
- 单表查询速度极快(利用并行查询)，但是 Join 效率很低
  - 通常可以通过冗余宽表的方式避免 join
- 多线程并行查询，可以将 CPU 打满，对 CPU 消耗较大，QPS 较低
- 文档：https://clickhouse.com/docs/en/
  - 虽然有中文版，但是翻译时会丢失一些示例导致难以理解，不如直接看英文版
- 集成引擎
  实现将其他数据源集成到 ClickHouse，如 MySQL/Kafka
- 表引擎
  建表时必须指定表引擎类型，不同表引擎提供不同的特性(数据源、存储方式、查询方式、附加功能)
  - TinyLog
    一般用于测试。
  - Memory
    全部数据放在内存，一般用于测试或数据较少的情况。
  - MergeTree
    是一大类引擎
- 数据类型
  - 整形
  - 字符串
  - 日期时间
  - 枚举
  - array
    - 一般不支持多维数组
- 关于主键索引`primary key`
  **主键是不保证唯一的**，要由使用者自己维护
- 关于排序`order by`
  - 在分区内部保证有序
  - MergeTree 表引擎，`order by`是**必须的**
- 分区字段，paritition by
  分区的目的是限制一次查询的范围，比如可以按日期的天进行分区
  - 建表时可以指定分区字段，写入时会根据字段值自动分区，读取时可以加分区限定
- 分片字段，shard key
  分片的目的是多个节点并行读取，避免单机瓶颈。shard key 使数据能够经过 hash 尽量分散到不同节点上。
- 关于 client
  - client 查询时，可以看到分区内数据情况
- 本机数据保存目录 `/var/lib/clickhouse`
  - `metadata`
    保存`xxx.sql`，是建表语句
  - `data`
    数据目录
- replication
  ClickHouse 可以依赖 ZooKeeper 进行配置，每个表可以设置自己的 replica
