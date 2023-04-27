---
layout: post
title: "Clickhouse SQL"
date: 2022-11-30 22:00:00 +0800
tags: ClickHouse
---

- Linux 和 macOS 安装 ch：`curl https://clickhouse.com/ | sh`

  - 启动 server `./clickhouse server`
  - 启动 client `./clickhouse client`

- 创建表

```SQL
 CREATE TABLE test_db.test_table_local on cluster test_cluster
(
    `name` varchar(50)
)
ENGINE = MergeTree
ORDER BY tuple();
```

- 创建分布式表

```SQL
create TABLE test_db.test_table_dist on cluster test_cluster as test_db.test_table_local
      ENGINE = Distributed("test_cluster", "test_db", "test_table_dist", rand());
```

- 查询执行的历史 SQL

```SQL
select query_id, query, event_time, exception from system.query_log where query like '%test_table%' order by event_time desc limit 20;
```

- `system.parts`
  分区情况

- `select * from test_db.test_table_local`
  查询分布式表中当前节点的 local 表

- `select * from test_db.test_table`
  查询分布式表的全部节点数据

- 注意查询时，字符串值要以`'`包围

# 函数

## 表函数

- `cluster('cluster_name', db.table[, sharding_key])`
  `SELECT * FROM cluster('{cluster}', default.example_table);`
  用于对一个集群中多个节点同时执行查询操作的函数。集群列表定义在"remote_servers"中。

## 数组函数

- `hasAny(target_array, ['str1', 'str2'])`
  - 判断`Nullable(Array(String))`类型的数组中是否包含目标字符串
