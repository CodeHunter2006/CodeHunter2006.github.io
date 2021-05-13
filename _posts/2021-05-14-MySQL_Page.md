---
layout: post
title: "MySQL Page存储结构"
date: 2021-05-14 23:00:00 +0800
tags: MySQL
---

![InnoDB Store](/assets/images/2021-05-14-MySQL_Page_1.jpg)
为了充分利用磁盘 16KB 的读取缓冲区容量，MySQL 设计了一套结构，了解底层结构后应用中才能避免踩坑。

# 存储结构

MySQL InnoDB 的每个表是独立存储的文件，其结构嵌套关系是：
Tablespace(表空间)->Table(表)->Segment(段)->Extent(区)->Page(页)

- InnoDB 存储包括三类**表空间**：

  - 系统表空间：
    主要存储 MySQL 内部的数据字典数据，如 information_schema 下的数据。
  - 用户表空间：
    当开启 innodb_file_per_table=1 时(从 MySQL5.6 开始设为默认值)，数据表从系统表空间独立出来存储在以 table_name.ibd 命令的数据文件中，
    结构信息存储在 table_name.frm 文件中。
  - Undo 表空间：
    存储 Undo 信息，如快照一致读和 flashback 都是利用 undo 信息。

![Disk](/assets/images/2021-05-14-MySQL_Page_2.png)
一个表文件(如 table_name.ibd)会被分为多个**段**，每个段与一个索引相关联。段会自动伸长或收缩。
段的下一级是**区**，一个区只会保存在一个段中，容量为 1MB。
区的下一级是**页**，默认为**16KB**，正好和磁盘一次读取的缓冲区容量一致。
一个区最多可以包含 64 页。InnoDB 规定一页至少容纳两行，所以行大小限制为**8000B**。
页的下一级是**行**，一页可以包含`2~n`行，每行容量由表结构设计时定义。

![B+ Node](/assets/images/2021-05-14-MySQL_Page_3.png)
一页对应于 B+树的一个结点，在 MySQL 中结点叫 **INodes**。
每个 INode 页由前面的**页内索引**和后面的数据组成，页内索引是顺序的，可以加快页内的查询。
每个 B+树都有一个叫做**根结点**的入口结点，根结点包含：索引 ID、INodes 数量等信息；

**InnoDB 的磁盘操作都是以页为单位进行的**
下面着重说明页内数据结构：
![Page](/assets/images/2021-05-14-MySQL_Page_4.png)
页可以被填充满、也可以只放两条数据。行记录按照主键排序。
**MERGE_THRESHOLD**是每个页的重要属性，用于页合并判断，默认值为 50%。
每行数据大小可能有不同。当插入新数据时，如果有空位则插入，如果已满，则插入下一页。
对于叶子结点页，是一个双向链表结点，有前后页的指针，用于范围查找。

## 页合并

**删除**行时，不会真正删除，而是做一个标记。
当一个页删除的足够多，低于 MERGE_THRESHOLD 时，InnoDB 查找相邻的页，尝试合并。
合并后，合并前后两个页中，后面的页会变为空页，可用于新数据。
当我们**更新**一条数据，由于新记录的大小使页面容量低于 MERGE_THRESHOLD 时，也会触发合并。
如果合并成功，在 INFORMATION_SCHEMA.INNODB_METRICS 表中的 index_page_merge_successful 指标将会增加。

## 页分裂

如果**插入**新数据时，当前页已满，并且后面的页也满了，则发生分裂。
在中间创建一个新页，将前页中高于 MERGE_THRESHOLD 的数据**迁移**到新页，然后将新增数据加到新页。
这种分裂会使页的序号发生**页错位**，比如在 7 8 页中插入一个新增的 9 页，那么 9 的物理位置实际上是在 7 8 之后，但是 B+树逻辑是在 7 8 之间。
同样在 INFORMATION_SCHEMA.INNODB_METRICS 表中记录了页分裂的次数 index_page_splits 和 index_page_reorg_attempts/successful 指标。
更新数据同样可能导致也分裂。一旦发生页错位，将其回收的唯一方法是相邻页容量降至 MERGE_THRESHOLD 以下，然后分裂页被迁移走。
另一种方法是执行**OPTIMIZE TABlE**碎片回收，但是这个 DML 操作将导致整个 MySQL 处于**不可用**状态，如果数据量大将无法接受。

在页分裂或合并时，会对索引树加一把**全局排他锁**，在繁忙的系统会造成问题。如果没有分裂或合并，那这个锁一般是共享锁，并且只加在页上。

# 常见问题和应对方法

- 是否通过删除数据减少容量？
  如上所述，即使执行了删除，实际上也是做了个标记，没有减少容量。如果执行碎片回收，会长时间卡死数据库，通常是不可接受的；
  删除数据也会引起更多的页合并(删除)、页分裂(删除后新增)，导致并发性能下降。

- 要设置自增整型 ID 作为主键
  由于数据是伴随主键保存的，如果主键是随机值，会导致频繁发生页分裂，会导致性能骤降。自增整型 ID 作为主键，可以减少发生分裂的概率。

- 最佳实践：表模板
  如下，对每个表都增加下面四个字段，可以方便的增加过滤条件

```SQL
`id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
`is_deleted` tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否逻辑删除：0：未删除，1：已删除',
`create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
`update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间'
```

- 最佳实践：归档数据
  可以定期进行归档，删除无效数据
  1. 创建归档表(不包含 is_delete 字段)
  2. 按时间过滤旧数据，在低峰时插入归档表(忽略被标记删除的数据)
  3. 创建临时表
  4. 按时间过滤把要保留的插入临时表
  5. 交换临时表和旧表，重命名
