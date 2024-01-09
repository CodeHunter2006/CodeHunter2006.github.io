---
layout: post
title: "Clickhouse SQL"
date: 2022-11-30 22:00:00 +0800
tags: ClickHouse
---

- Linux 和 macOS 安装 ch：`curl https://clickhouse.com/ | sh`

  - 启动 server `./clickhouse server`
  - 启动 client `./clickhouse client`

- `on cluster xxx`
  在 clickhouse 中的特殊语法。执行 DDL 时加上`on cluster`可以自动在集群所有节点执行该 DDL。
  其原理是客户端把请求打到某个初始节点执行时，该节点会根据配置中的集群拓扑结构把请求分别打到所有节点(所有主备)执行，分别执行完毕返回到初始节点后，初始节点再把结果返回给客户端。
  查询时，只要使用了分布式表，就会以同样的请求路径执行查询。

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
-- Distributed 参数：集群名、库名、对应的 local 表名，
create TABLE test_db.test_table_dist on cluster test_cluster as test_db.test_table_local
      ENGINE = Distributed("test_cluster", "test_db", "test_table_dist_local", rand());
```

- 利用 AS 和 LIKE 建表

```SQL
-- AS 表示利用原表的元数据建表，后面可选项 ENGINE 可以修改表引擎
CREATE TABLE db1.new_table_name AS db2.old_table_name ENGINE = Distributed("test_cluster", "test_db", "test_table_dist", rand());
```

- 查询执行的历史 SQL

```SQL
select query_id, query, event_time, exception from system.query_log where query like '%test_table%' order by event_time desc limit 20;
```

- `system.tables`
  表情况

- `system.parts`
  分区情况

- `system.kafka_tables`
  kafka 消费表情况

- `select * from test_db.test_table_local`
  查询分布式表中当前节点的 local 表

- `select * from test_db.test_table`
  查询分布式表的全部节点数据

- 注意查询时，字符串值要以`'`包围

- 导入文件
  `clickhouse-client --format_csv_delimiter=";" -q 'insert into table_name format CSV' < test.csv`

- 导出文件
  `select * from table_name limit 0,100 into outfile 'tmp.csv' format CSV;`

## INSERT INTO

```SQL
-- 一次插入多行数据，每行数据用 () 包裹，括号间用 , 分割，括号内列值以 , 分割
INSERT INTO table_name FORMAT VALUES (1, '1'),(2, '2')
```

# 函数

## 表函数

- `cluster('cluster_name', db.table[, sharding_key])`
  `SELECT * FROM cluster('{cluster}', default.example_table);`
  用于对一个集群中多个节点同时执行查询操作的函数。集群列表定义在"remote_servers"中。

  - 第一个参数集群名
  - 第二个参数是想查的库表名
  - 第三个参数是 replica 编号，`(1)`-第一个，`(2)`-第二个...，默认为 `(0)`。
    - `(0)`-全部 replica，随机选一个。
    - `(1,2,3...)`可以同时填写多个，如果有的话。表示同时从这些 replica 查出结果。

- `host()`
  查询当前节点 ip

## 数组函数

- `hasAny(target_array, ['str1', 'str2'])`
  - 判断`Nullable(Array(String))`类型的数组中是否包含目标字符串
