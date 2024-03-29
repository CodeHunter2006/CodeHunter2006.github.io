---
layout: post
title: "MySQL Sharding"
date: 2020-04-25 23:00:00 +0800
tags: MySQL
---

为什么要分区分库分表？
通常 MySQL 的表支持到百万级时就开始产生性能问题，当达到千万级时就需要进行分区分库分表来提高性能。

# 引起性能下降的原因

- 数据量超过百万时，三层 B+树的索引已经开始出现检索性能下降，如果联表查询，性能会急剧下降。拆分后可以降低树高度
- InnoDB 在索引失效的情况下，行锁会退化为表锁，当数据量较大时，锁竞争的频率会越来越高，上下文切换会使性能急剧下降。拆分后可以减少锁冲突
- 一台服务器的资源(CPU、磁盘、内存、IO 等)是有限的，数据量、处理速度都将遭遇瓶颈。拆分后可部署到不同服务器
- 磁盘系统最大文件限制。拆分后可突破单文件限制

| 格式  | 最大分区 | 最大容量 | 最大文件 |
| ----- | -------- | -------- | -------- |
| FAT16 | 2GB      | -        | 2GB      |
| FAT32 | 32GB     | 2TGB     | 32GB     |
| NTFS  | 2TGB     | 2TGB     | 2TGB     |
| EXT3  | 2TGB     | 4TGB     | 4TGB     |

## 性能问题及建议

- 优化路线：正常建表->主从复制->垂直分库(不同表分开)->单库单表->垂直分表(切分字段)->分区(自动水平分表)->水平分表(代码逻辑复杂，尽量不用)
- 通常一个表的容量不要超过 500 万，如果字段特别少，不要超过 1000 万。否则就要拆成多个表。
- 分库分表要掌握好粒度，结合考虑"业务紧密程度"、"字段的读取频率"、"表的数据量范围"、"表的数据量增速"、"存量数据"、"增量数据"

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

- 什么是分库？
  分库又叫垂直切分，把原本存储于一个库的表拆分存储到多个库上，通常是将表按照功能模块、关系密切程度划分出来，部署到不同的库上。
  如果数据库是因为表太多而造成海量数据，并且项目的各项业务逻辑划分清晰、低耦合，那么规则简单明了、容易实施的首选就是分库。

  - 优点：实现简单，库与库之间界限分明，便于维护
  - 缺点：不利于频繁跨库操作，单表数据量大的问题解决不了

- 什么是分表？
  分表又叫水平切分，按照一定的业务规则或逻辑，将一个表的数据拆分成多份，分别存储在多个表结构中，这多个表可以存在一个或多个库中。分表又分为垂直分表和水平分表。
  分表的优点：解决分库的不足，但实现比较复杂，特别是分表规则、程序编写、后期拆分、移植维护都比较困难。

- 垂直分表
  将按照关系型数据库第三范式要求的在同一个表中的字段，人为划分为多个表(字段分割)。需要新增关联 ID 字段。

- 水平分表
  也被称为数据分片，把一个表按行进行切分，从而保证单表的容量不会太大，提升性能。这些表可以放在一个或多个数据库中。

  - 优点：
    - 对数据分片后，并发压力被分散到不同磁盘，提高了性能
  - 缺点：
    - 实现 join 等操作较复杂，会降低性能

- 如何理解**垂直**？
  无论是垂直分库还是垂直分表，拆分后的不同实例间的结构是不同的。

## 水平分表的实现

- 水平分表面临的一系列问题：

  - 全局主键生成
    无法使用数据库自增 ID，要使用**雪花算法**生成全局自增 ID
  - 跨节点排序/分组/分页/表关联等操作
    需要利用组件(库/代理)实现
  - 多数据源事务处理
    需要利用**分布式最终一致性协议(TCC/XA)**实现**分布式事务**
  - 切分策略
    要根据业务情况决定用什么字段进行拆分
  - 库节点路由
  - 表路由
  - 数据库扩容

- 水平分表有很多开源产品，但是除了 Fabric 大都不支持事务，而如果不用事务，完全可以使用 MongoDB。
- 通常有两种实现方式：
  - 库实现
    如 Fabric、JDBC 直连层(ShardingSphere、TDDL)，在驱动层实现，性能更高。
    - 优点：
      - 性能高
    - 缺点：
      - 对语言强依赖，如 JDBC 只能用于 Java
  - 代理实现
    如 MySQL Proxy、Mycat，在代理配置策略，客户端发送 SQL 在代理被转换为具体分库查询和综合结果返回；
    - 优点：
      - 可以跨语言
    - 缺点：
      - 由于存在中介进程，增加了网络延迟降低了性能
- 如果追求通用设计，往往难以找到合适的开源库，但是如果根据业务自己写具体分表实现代码，还是可行的

## 基本实现思路

1. 解析路由：根据业务功能/SQL 解析指定。获得要访问的数据源以及要访问的表
2. 分别在数据源和表上去执行功能
3. 如果涉及到返回结果集，需要做合并、排序、分页等
4. 如果需要事务，需要考虑使用分布式事务/自行实现两阶段提交/采用补偿性业务处理方式等

## 可实现的层面(Java 为例)

1. DAO(Data Access Object)层
2. Spring 数据访问封装层，介于 DAO 和 JDBC 之间
3. JDBC 驱动层
4. 代理服务器，介于应用服务器与数据库之间

## 为什么分库、分区、水平分表时不能使用自增主键 ？

- 场景：
  假设分为三个表，每个表要设置一个自增主键的起始值，这样相当于设定了每个表的自增主键范围。
- 问题：

  - 提前设定自增主键范围，如果设置范围较大，可能造成浪费，未来新增表时范围不足
  - 如果设置范围较小，可能会不够用，一旦满了会面临更多问题
  - 会产生**尾部热点**效应，集中于一个表产生大量并发，没有起到分散热点的作用

- 是否能用 UUID 替代？
  不能。虽然 UUID 确保不重复，但是由于 InnoDB 聚集索引的特点，如果 ID 不是顺序的，会造成大量索引分裂、性能急剧下降

- 方案：
  采用 SnowFlake 算法，生成分布式唯一 ID(1bit 符号位 + 41bit 毫秒时间戳 + 10bit 机器 ID + 12bit 序列号)。
  这样即保证了 ID 有序，又无需提前预留。
  由于 SnowFlake 算法通常时间匹配度较高，并且时间戳是在最高位(符号位通常丢弃)，所以可以利用**时间范围**进行数据查询。
- 问题：
  雪花算法生成 ID 要注意时间回拨带来的影响，如果允许一定的回拨，短时间内的数据可能乱序；如果允许较大的回拨，则无法信任时间戳范围；

# 应用场景

- 垂直分库
  多个业务共用一个数据库导致并发无法提高，所以执行垂直分库，将不同业务的表拆分到不同库。
  例如将"user_info"、"orders"、"goods"三张表拆分到不同数据库实例。

- 垂直分表
  之前设计的一张表，其中不同字段使用频率不同，比如"user_info"表中"user_name"和"password"访问频率较高，而"id_card"和"real_name"访问频率较低，
  可以将"user_info"表拆分为"user_base"和"user_info"两个表，用自增的"user_id"作为主键

- 水平拆分的字段选择
  "orders"表有"userID"和"datetime"两个字段，当"orders"由于并发和容量过大需要水平分表时，应该根据哪个字段？
  - 通常应该根据"userID"字段，这样可以快速查询一个用户的所有订单，这种查询频率较高。
    - 这时如果按日期查询，需要在多个表或库中利用中间件查询
  - 但是如果"按日期查询"的查询频率较高时，也可以按日期查询分表
    - 这样做的缺点是插入数据总是最新时间，所以插入请求会产生"尾部热点"效应，总在一个表中插入，造成瓶颈
