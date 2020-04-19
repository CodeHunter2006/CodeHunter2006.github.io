---
layout: post
title: "MySQL Slowlog Profiles"
date: 2020-04-18 23:00:00 +0800
tags: MySQL
---

# Slowlog

Slowlog(慢查询日志)是 MySQL 提供的一种日志记录，用于记录 MySQL 服务端响应时间超过阈值的 SQL。

- 用变量`long_query_time`设置慢查询阈值，默认 10s。
- 慢查询默认是关闭的，可以在开发时打开，上线后关闭。
- 查询状态：`show variables like '%slow_query_log%';`
- 临时开启(进程退出失效)：`set global slow_query_log = 1;`
- 永久开启：在`/etc/my.cnf`在`[mysqld]`增加配置`slow_query_log = 1`，重启 MySQL 服务
- 可以用`slow_query_log_file`变量设置慢查询日志输出位置
- 在客户端用命令设置了`long_query_time`后，要重新登录后生效。session 级的设置。
- 通过变量查询是否有慢查询：`show global status like '%slow_querys%'`，如果不等于 0，则到 log 中查看具体慢查询语句

## mysqldumpslow

方便查看慢查询 log 的工具，安装 MySQL 时自带的，直接执行`mysqldumpslow 参数 /.../slow.log`即可。

- `-s` 排序方式
  - `r` 返回记录数最多的
  - `c` 访问次数最多的
- `-r` 逆序
- `-l` 锁定时间
- `-g` 正则匹配模式
- `-t` 显示的结果条数

# profiles

profiles 可以记录所有执行的 SQL 查询语句，用于分析，默认是关闭的。

- `show profiles;` 显示历史 SQL 语句，同时显示执行序号(query_id)、执行耗时(duration)。
- `show variables like '%profiling%';` 显示 profiles 开关状态，默认关闭。
- `set global profiling = 1;`/`set global profiling = on;` 打开开关

上面显示的信息比较概要，对于 duration 究竟在哪些地方(IO/CPU/内存)耗时了，没有详细列出。
可以用下面语句进行 SQL 诊断：

- `show profile all for query query_id;` 显示所有详细耗时，其中`query_id`是上一步查到的
- `show profile cpu, block io for query query_id;` 显示指定的耗时

# 全局查询日志(作为参考)

记录开启之后的全部 SQL 语句，默认是关闭的。通常只在开发、调优阶段打开日志，在上线时关闭。
开启后，log 会被记录到`mysql.general_log`表中。
由于全局查询日志只有 SQL 语句，没有执行时间，所以只能作为前面优化查询的参考。

- `show variables like '%general_log%'` 查看开关
- `set global general_log = 1;` 打开开关
- `set global log_output='table'` 设置全局日志输出到表
- `set global log_output='file'` 设置日志输出到文件
- `set global general_log_file='/tmp/tmp.log'` 设置输出的日志文件的路径
