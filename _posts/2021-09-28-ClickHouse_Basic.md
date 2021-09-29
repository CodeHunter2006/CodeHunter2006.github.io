---
layout: post
title: "Click House 基础"
date: 2021-09-28 22:00:00 +0800
tags: ClickHouse
---

![ClickHouse](/assets/images/2021-09-28-ClickHouse_Basic_1.png)

- 由俄罗斯 xxx 在 xxx 开源
- OLAP
- 列存储，适合存储宽表数据
- 单表查询速度极快(利用并行查询)，但是 Join 效率很低
- 多线程并行查询，可以将 CPU 打满，对 CPU 消耗较大，QPS 较低
- 支持 SQL
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
- 可指定分区`partition by`
  在查询时
- 关于 client
  - client 查询时，可以看到分区内数据情况
- 本机数据保存目录 `/var/lib/clickhouse`
  - `metadata`
    保存`xxx.sql`，是建表语句
  - `data`
    数据目录
- replication
  ClickHouse 可以依赖 ZooKeeper 进行配置，每个表可以设置自己的 replica
