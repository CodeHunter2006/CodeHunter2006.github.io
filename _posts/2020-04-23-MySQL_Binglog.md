---
layout: post
title: "MySQL Binlog"
date: 2020-04-25 23:00:00 +0800
tags: MySQL
---

![binlog](/assets/images/2020-04-23-MySQL_Binglog_1.jpeg)
binlog(binary log)二进制日志，是 MySQL 最重要的日志，记录了所有 DDL 和 DML 语句(除了查询语句)，以事务形式记录并包含耗时。

# binlog 的使用场景

1. **主从复制**
   mysql replication 在 master 端开启 binlog，master 将二进制日志传递给 slave 端进行同步，是数据一致
2. **数据回复**
   通过 binlog 记录历史变化，用 mysqlbinlog 工具来恢复数据

- 一般说开启 binlog 日志大概会有 1%的性能损耗
- 在进行增量备份或主从复制时，都需要开启 binlog 日志，最好跟数据目录设置到不同的磁盘分区，可以降低 io 等待，提升性能；并且在磁盘发生故障的时候，避免在同一分区导致数据丢失

# binlog 三种格式

1. **STATEMENT**
   基于 SQL 语句的复制(statement-based replication, SBR), 每一条会修改数据的 SQL 会记录到 binlog 中。
   - 优点：日志较少、减少磁盘 IO，性能较高
   - 缺点：某些情况下会导致 master-slave 数据不一致(如 sleep()，last_insert_id()，user-defined functions(udf))
2. **ROW**
   基于行的复制(row-based replication, RBR)，记录被修改的数据的变更内容。
   - 优点：不会出现调用函数或调用时机导致的不一致问题
   - 缺点：会产生大量日志，尤其是 alter table 时会日志暴涨
3. **MIXED**
   上面两种模式的混合，一般的复制使用 STATEMENT 模式保存，对 STATEMENT 无法保证一致性的保存被修改的数据的变更内容。

# binlog 配置

my.cnf 中[mysqld]字段

- `binlog_format = mixed` 日志格式
- `log-bin = /data/mysql/logs/mysql-bin.log` 日志路径
- `expire_logs_days = 7` 日志清理时间
- `max_binlog_size = 100m` 每个日志文件大小
- `binlog_cache_size = 4m` 缓存大小
- `max_binlog_cache_size = 512m` 最大缓存大小

# mysqlbinlog 常用选项

- `--start-datetime`读取时间戳大于等于指定时间的日志
- `--stop-datetime`读取时间戳小于等于指定时间的日志
- `--start-position`以指定事件位置为开始
- `--stop-position`以指定事件位置为截止
