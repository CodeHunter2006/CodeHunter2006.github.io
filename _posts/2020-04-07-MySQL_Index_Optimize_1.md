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
