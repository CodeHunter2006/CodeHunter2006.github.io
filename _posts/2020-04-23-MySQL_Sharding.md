---
layout: post
title: "MySQL Sharding"
date: 2020-04-25 23:00:00 +0800
tags: MySQL
---

为什么要分区分库分表？
通常 MySQL 的表支持到百万级时就开始产生性能问题，当达到千万级时就需要进行分区分库分表来提高性能。

# 引起性能下降的原因

- 数据量超过百万时，三层 B+树的索引已经开始出现检索性能下降，如果联表查询，性能会急剧下降
- InnoDB 在索引失效的情况下，行锁会退化为表锁，当数据量较大时，锁竞争的频率会越来越高，上下文切换会使性能急剧下降
- 一台服务器的资源(CPU、磁盘、内存、IO 等)是有限的，数据量、处理速度都将遭遇瓶颈
- 磁盘系统最大文件限制
  |格式|最大分区|最大容量|最大文件|
  |-|-|-|-|
  |FAT16|2GB|-|2GB|
  |FAT32|32GB|2TGB|32GB|
  |NTFS|2TGB|2TGB|2TGB|
  |EXT3|2TGB|4TGB|4TGB|

## 性能问题及建议

- 优化路线：正常建表->主从复制->垂直分库(不同表分开)->单库单表->垂直分表(切分字段)->分区(自动水平分表)->水平分表(代码逻辑复杂，尽量不用)
- 通常一个表的容量不要超过 500 万，如果字段特别少，不要超过 1000 万。否则就要拆成多个表。

# 分区

将一个表分解成多个区块进行操作和保存，从而降低每次操作的数据量，提高性能。对应用来说，分区是透明的，从逻辑上看只是一个表，在物理上这个表可能是由多个物理分区组成，每个分区是一个独立的文件，可以独立处理。
可以通过拆分分区逐步提高总的表的数据量，如果还是不够，则可以合并分区，然后用分表的方式处理。

## 分区的功能

1. 进行逻辑数据分割，分割后的数据有多个不同的物理文件路径
2. 可以存储更多数据，突破系统单个文件最大限制
3. 提升性能，提高每个分区的读写速度，提高分区范围查询速度
4. 通过删除分区来快速删除数据
5. 可以跨多个磁盘来分散数据查询，从而提高磁盘 I/O 的性能
6. 涉及 SUM()、COUNT()这些聚合函数时，可以进行并行处理
7. 可以备份和恢复独立分区

## 可支持分区的引擎

MyISAM、InnoDB 都支持分区。同一个表的所有分区必须用同一种引擎。

- `show plugins;` 列表中显示`partition Active`表示支持分区
- 5.5 版本以上支持非整形(字符串等)分区字段
- 如果分区列值是`NULL`，可能会引起分配问题，所以应该设置为`NOT NULL`

## 分区类型

1. RANGE 分区
   给予一个给定连续区间的列值，把多行分配给分区
2. LIST 分区
   每个分区按照指定的离散值进行分区。适合针对类型分区的字段
   - 插入超出指定范围的值时，会报错
3. HASH 分区
   用用户指定的表达式产生非负 HASH 值选择分区。在指定数量的分区内数据较为分散，适合高并发
   - 由于每次 CRUD 都要进行一次运算，批量操作时可能会有大量计算，所以复杂的表达式可能会引起性能问题
   - 表达式可以是多列运算也可以是单列，但是通常用单列，避免性能问题
   - 哈希函数的运算结果最好随列值进行一致性的增大或减小，这样之后可以针对性的进行分区优化。例如用时间字段作为 Hash 函数时，取"时分秒"比"年月日"更为合适。
   - 通常默认的 Hash 函数是取余，也可以用`PARTITION BY LINEAR`利用线性 Hash 函数运算。优点：增加、删除、合并、拆分分区时，更加方便，有利于处理数据量极大的表(拆分)。缺点：各个分区间的数据分布不太均衡。
4. KEY 分区
   类似 HASH 分区，不允许用户自定义表达式，由 MySQL 提供 Hash 函数。KEY 分区支持除 blob 或 text 之外的各种数据类型。
   - 不指定分区键时，默认使用主键或唯一键。如果没有主键或唯一键，必须指定分区键。

## 子分区

分区表中每个分区再次分割，适合保存非常大量的数据。

- 子分区可以使用 HASH 分区或 KEY 分区，也被称为复合分区
- 每个分区必须有相同数量的子分区
- 如果在一个分区表上的任何分区上使用`SUBPARTITION`来明确定义任何子分区，那么就必须定义所有的子分区
- 每个`SUBPARTITION`子句必须包括(至少)子分区的一个名字
- 在每个分区内，子分区的名字必须是唯一的，目前在整个表中，也要保持唯一
- 设定子分区时，可以指定磁盘分配

## 相关语句

```SQL
-- 创建表，设定分区，Range分区
CREATE TABLE ... ()
PARTITION BY RANGE (uuid) (
   PARTITION p0 VALUES LESS THAN (10),
   PARTITION p1 VALUES LESS THAN (100),
   PARTITION p2 VALUES LESS THAN MAXVALUE
);

-- List分区
PARTITION BY LIST (uuid) (
   PARTITION p0 VALUES IN (1, 3, 5)
   PARTITION p1 VALUES IN (2, 4, 6)
)；

-- Hash分区，这里的表达式是直接取值
PARTITION BY HASH (uuid)
PARTITIONS 3;

-- Key分区
PARTITION BY LINEAR KEY (uuid)
PARTITIONS 3;

-- 子分区，自动命名
PARTITION BY RANGE (YEAR(registerTime))
   SUBPARTITION BY HASH(TO_DAYS(registerTime))
   SUBPARTITIONS 2
(
   PARTITION p0 VALUES LESS THAN (2010),
   PARTITION p1 VALUES LESS THAN (2020),
   PARTITION p2 VALUES LESS THAN MAXVALUE
);

-- 子分区，自定义子分区
PARTITION BY RANGE (YEAR(registerTime))
   SUBPARTITION BY HASH(TO_DAYS(registerTime))
(
   PARTITION p0 VALUES LESS THAN (2010)(
      SUBPARTITION s0,
      SUBPARTITION s1,
   ),
   PARTITION p1 VALUES LESS THAN (2020)(
      SUBPARTITION s2,
      SUBPARTITION s3,
   ),
   PARTITION p2 VALUES LESS THAN MAXVALUE(
      SUBPARTITION s4,
      SUBPARTITION s5,
   )
);

-- 查看分区信息
SELECT * FROM information_schema.partitions WHERE table_schema='arch1' AND table_name='tbl_users'  \G;

-- 查看指定分区的数据
SELECT * FROM tbl_users partition(p0);

-- 分析分区操作情况
EXPLAIN PARTITIONS SELECT ...
```

## 分区管理

添加、删除、重新定义、合并、拆分等管理操作

- 删除
  - 删除一个分区，同时也会删除该分区中所有数据
  - 如果是 List 分区，对应的列值也会被删除，所以该列值无法再插入
- 添加
  - 对于 Range 分区的表，只可以添加新的分区到分区列表的高端(超出原有范围)
  - 对于 List 分区的表，不能添加已经包含在现有分区值列表中的任意值
- 重新定义分区(保留数据)
  - 拆分
  - 合并
- 删除所有分区(保留数据)
- 对于 HASH 和 KEY 分区的管理
  只能增加和减少分区的数量
- 其他管理
  - 重建分区
    相当于先删除保存在分区中的所有记录，然后重新插入它们，可用于整理分区碎片
  - 优化分区
    如果从分区中删除了大量的行，或者对一个带有可变长度的行(VARCHAR/BLOB/TEXT)做了许多修改，可以使用`OPTIMIZE PARTITION`来回收没有使用的空间，并整理分区数据的文件碎片。优化 = 分析->检查->修补->优化
  - 分析分区
    读取并保存分区的键分布
  - 检查分区
    检查分区中的数据或索引是否已经被破坏
  - 修补分区
    修补被破坏的分区

```SQL
-- 删除分区
ALTER TABLE tbl_users DROP PARTITION p0;

-- 添加分区
ALTER TABLE tbl_users ADD PARTITION(
   PARTITION p1 VALUES LESS THAN(100)
);

-- 重新定义分区
ALTER TABLE tbl_name REORGANIZE PARTITION partition_list INTO (partition_definitions);

-- 重新定义分区，拆分分区
ALTER TABLE tbl_users REORGANIZE PARTITION p1 INTO (
   PARTITION s0 VALUES LESS THAN(10),
   PARTITION s1 VALUES LESS THAN(20)
);

-- 重新定义分区，合并分区
ALTER TABLE tbl_users REORGANIZE PARTITION s0, s1 INTO (
   PARTITION p0 VALUES IN(1,3,5)
);

-- 删除所有分区，保留数据
ALTER TABLE tbl_users REMOVE PARTITIONING;

-- 减少 HASH/KEY 分区数量，数量减 2
ALTER TABLE tbl_users COALESCE PARTITION 2;

-- 增加 HASH/KEY 分区数量，数量增 2
ALTER TABLE tbl_users ADD PARTITION PARTITIONS 2;

-- 重建分区
ALTER TABLE tbl_users REBUILD PARTITION p2, p3;

-- 优化分区
ALTER TABLE tbl_users OPTIMIZE PARTITION p2, p3;

-- 分析分区
ALTER TABLE tbl_users ANALIZE PARTITION p2, p3;

-- 检查分区
ALTER TABLE tbl_users CHECK PARTITION p2, p3;

-- 修补分区
ALTER TABLE tbl_users REPAIR PARTITION p2, p3;
```

## 分区注意事项

- 最大分区数目不能超过 1024，一般建议单表的分区数不要超过 150 个
- 如果含有唯一索引或主键，则分区列必须包含在所有的唯一索引或主键内
- 分区不支持外键
- 不支持全文索引，对分区表的分区键创建索引，那么这个索引也将被分区
- 按日期进行分区很合适，因为很多日期函数可以使用。但是对于字符串来说，没有太多合适的分区函数。
- 只有 RANGE 和 LIST 分区能进行子分区，HASH 和 KEY 分区不能进行子分区
- 通常一级分区就够使用了，不到万不得已不要再划分子分区。而往往直接就按照分表设计了
- 临时表不能被分区
- 分区表对于单条记录的查询没有优势

# 分库分表
