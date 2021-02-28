---
layout: post
title: "MySQL基本结构、常用引擎"
date: 2019-07-18 10:00:00 +0800
tags: MySQL
---

# MySQL 数据库基本结构

![MySQL structure](/assets/images/2019-07-18-MySQL_structure_1.jpg)

如图，MySQL 是一个 C/S 架构的数据库服务器。具有三层结构：连接层、服务层、存储引擎层。

### 连接层

- 提供 TCP 连接
- 进行用户安全验证

### 服务层

- 解析器，解析用户输入的 SQL，解析成多种执行计划
- 优化器，优化 SQL 语句和执行计划，选择一个最优的执行计划(考虑 CPU、IO、Memory)
- 执行器，按照优化器的选择执行 SQL 语句，获取结果数据

- **QueryCache 机制**
  如果客户端发来的多次 query 请求完全相同(精确到字母大小写)，那么 MySQL 会直接将缓存结果返回，不进行解析、优化、执行过程。

### 存储引擎层(常用 InnoDB/MyISAM)

- 按照 SQL 的需要，把数据以特定的数据结构存储与文件中
- 提供锁、事务、缓存机制

# InnoDB 和 MyISAM 引擎的差别

MyISAM 是 MySQL 自带的存储引擎，但是它对高并发支持的不好。而第三方的 InnoDB 引擎，提供了较好的并发性。随着互联网对高并发需求的不断增加，MySQL 结合 InnoDB 迎来了快速发展。(5.1 之前默认是 MyIsam，之后默认是 InnoDB)

| 特点             | InnoDB                                                   | MyISAM                                               |
| :--------------- | -------------------------------------------------------- | ---------------------------------------------------- |
| 主要应用方向     | 面向 OLTP（Online Transaction Process，在线事物处理）    | 面向 OLAP（Online Analytical Process，在线分析处理） |
| 对事务的支持     | 支持事务                                                 | 不支持事务                                           |
| 对锁的支持       | 表锁、行锁                                               | 表锁                                                 |
| 对外键的支持     | 支持外键                                                 | 不支持                                               |
| 对高并发的支持   | 支持高并发                                               | 不支持，并发下性能急剧下降                           |
| 对全文索引的支持 | 不支持                                                   | 支持全文索引                                         |
| 对全表总数的统计 | 需要实时计算                                             | 可直接查询总数记录                                   |
| 存储空间大小     | 相比 MyISAM 存储空间较大，没有压缩、缓存、索引要占据容量 | MyISAM 支持数据压缩，容量较小                        |
| 单个表容量限制   | 单个表容量受操作系统限制，一般 2GB                       | 无容量限制                                           |

从上面的对比可以看出，InnoDB 在锁、事务方面都对高并发有较好的支持，我们通常默认使用 InnoDB。后面文章中默认以 InnoDB 为引擎。

# DML/DDL

Mysql 是 C/S 模式，所以 client 端输入的 SQL 需要执行 commit 统一提交到 server 端处理再返回结果。Mysql client 端默认是自动带 commit 的。

- DML(Data Manipulation Language)
  就是我们通常的增删改查语句，后面需要 commit。
- DDL(Data Definition Language)
  是用来管理数据库的语句，自动 commit，不能回滚。
