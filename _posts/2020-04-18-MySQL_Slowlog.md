---
layout: post
title: "MySQL Slowlog"
date: 2020-04-18 23:00:00 +0800
tags: MySQL
---

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
