---
layout: post
title: "MySQL 索引优化1"
date: 2020-04-07 23:00:00 +0800
tags: MySQL
---

# Mysql 查询 SQL 或索引设置不当时会有哪些问题？

- 性能低
- 执行时间长
- 等待时间长
- SQL 语句结构差(连接查询)
- 索引失效
- 服务器参数设置不合理(缓冲、线程)

# SQL 的执行过程

从我们输入的 SQL 到最终得到结果，进行了下面步骤：写 SQL->解析器->优化器->执行器

- 所以可能会发生优化其篡改语句的情况

## 编写过程

```SQL
select distinct...from...join...on...where...group by...having...order by...limit
```

## 解析过程

```SQL
from...on...join...where...group by...having...select distinct...order by...limit
```

由于编写和执行存在很大差异，所以我们要按照最终执行的过程去做优化

# SQL 的优化

SQL 的优化，主要是围绕索引进行

## 索引

### 索引的缺点

- 降低 IO 操作次数，通过 B+树结构减少比较次数和深度，三层 B+树可存放上百万
- 降低 CPU 使用率，索引是有序的，无需排序

### 索引的缺点

- 索引本身容量很大，可以存放在内存/硬盘中(通常是硬盘)
- 索引只适合容量小的字段，大容量字段做索引成本太大
- 索引可以提高查询效率，但是增、删、改的成本很高
- 索引不适合很少使用的列，在 where 中被使用时才能起作用

### 索引分类

- 单值索引 单列索引，一个表可以有多个单值索引
- 唯一索引 是单值索引，不能重复。比如: id
- 复合索引 多个列顺序组合构成。复合索引查找时无需全部命中。
- 主键索引 唯一索引+非 NULL

## expain

在 SQL 前加上`explain`可以查询执行计划，模拟 SQL 优化器执行 SQL 语句，从而分析自己的 SQL 被执行的情况。

### 显示的字段及其意义

- id 优先级
  - id 值相同，从上向下顺序执行; 值不同，数值越大越优先执行;
  - 多表联查，id 值相同，优化器会把数据量小的表优先查询，这样做笛卡尔积时中间结果数据量更小
  - 多表联查可以转换为嵌套子查询，id 值不同，内层查询优先级更高
- select_type 查询类型
  - SIMPLE 简单查询(不包含子查询、union)
  - PIMARY 主查询(最外层查询)
  - SUBQUERY 子查询(嵌套在里面的查询)
  - DERIVED/UNION 衍生查询，用到了临时表
    - 在 from 子查询中只有一张表，则该表是衍生表
    - 在 from 子查询中如果 t1 union t2，则 t1 是衍生表，t2 是 UNION 表
- table 表名
- type 遍历时对索引的利用类型，按性能排序：
  system > const > eq_ref > ref > range > index > all
  - type 中 system、const 是理想情况，实际上能达到 ref、range 就可以了
  - 对 type 优化的前提是建立过索引
  - system 只有一条数据的系统表或衍生表只有一条数据的主查询
  - const 仅能查到一条数据的 SQL，用于 Primary key 或 Unique index
  - eq_ref 唯一性索引，对于每个索引键的查询，有且仅有一条数据返回。常见于唯一索引或主键索引
  - ref 非唯一性索引，通常可以命中
  - range 范围查询，`between and, >, <, >=`都能命中，`in`有时会失效
  - index 索引命中，同时可能伴随着最后一次回查
  - all 查询全部表中的数据, 没有任何优化
- possible_keys 预测用到的索引，NULL 为没有使用索引
- key 实际用到的索引
- key_len 实际使用索引的字段长度(字节数)之和，可以通过长度反推复合索引用是否被完全使用
  - 根据不同的字符集、编码，字符串索引命中长度可能不同
  - 可以为空的字符串，len 会多 1，使用一个字节标识可以为 NULL
  - varchar 相比 char，len 会多 2，用来表示字符串长度
- ref 指明当前表参照的字段
  - const 引用的是一个常量
- rows 实际通过索引查到的数据个数
- extra 额外信息
  - using filesort 性能较差，需要额外一次查询或排序，常见于 order by 语句，查找(where)和排序(order by)不是同一字段
    - 对于单索引，可以 where 哪些字段就 order by 哪些字段，这样就不需要多查一次
    - 对于复合索引，按照"最佳左前缀"原则，where 和 order by 按照顺序使用，不要跨列或无序使用
  - using temporary 性能较差，用到了临时表，常见于 group by 语句，where 和 group by 不是同一字段
  - using index 性能较好，索引覆盖，不读取原文件(回表查询)只读取索引。这里是指查询阶段不需要回表，由于 select 字段不同导致最后一次查询需要回表不算在内
  - using where 即需要索引查询也需要回表查询
  - impossible where , where 子句永远为 false，SQL 语句里有自相矛盾
  - using join buffer 优化器修改了 SQL，在连表查询时增加了连接缓存
