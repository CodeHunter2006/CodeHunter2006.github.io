---
layout: post
title:  "MySQL数据文件的存储核心——PRIMARY KEY"
date:   2019-07-19 10:00:00 +0800
tags: MySQL
---

![Hard Disk](/assets/images/2019-07-19-MySQL_primary_key_disk_1.jpg)
MySQL的数据是存储在硬盘上的，由于硬盘的机械结构，每次读写要等到磁盘的扇区旋转到合适的位置时才能进行，所以速度相比内存(纯电子电路)慢两个数量级(100倍以上)。

基于这种速度差异，在设计数据库时就要遵守一个原则——尽量少的进行硬盘操作，而主键的设计正好符合这个原则。下面以InnoDB为引擎，解释主键的作用。

# MySQL数据库主键
![B+Tree Storage](/assets/images/2019-07-19-MySQL_primary_key_disk_2.png)

InnoDB需要主键(或主键替代者)来作为数据存储的聚集索引，主键本身也是一个索引(unique not null increase)，所以也采用B+树，具体B+树的结构在索引章节讲解。

由于硬盘每次的读取容量是一个固定值(4K 8K 16K)，也就是在连续的硬盘上读取1K或15K都是一次操作，耗费的时间相同。所以InnoDB中的数据是以页(page)为单位存储的，不同页中的数据在硬盘上可以是无序的，而同一个页内的数据是有序的。

查看页大小
```
show variables like 'innodb_page_size'
```

如果我们定义了主键(PRIMARY KEY)，那么InnoDB会将主键作为聚集索引；如果没有显示定义主键，InnoDB会自动找一个unique not null索引作为主键；如果没有找到，InnoDB会为该表内置一个6字节的自增ROWID作为隐含的聚集索引。

可以看出，无论如何数据库都需要一个主键，所以最好我们最好自己定义，避免默认创建的主键引起其他问题。

# 主键使用的一些注意点
在一个页中，数据是有序的，当向页中间插入数据时，就会进行数据移动；如果数据量超过页的容量，则会创建一个新页存储，这就是页分裂。

如果频繁对已经存在的页进行插入、删除记录，就会发生很多移动、分裂，形成大量的碎片。原本设计每个页16K是为了减少IO，而大量碎片的出现会导致IO效率降低。

例如上一节的说明中，如果没有自定义主键，但是找到一个"unique not null index"，那么新数据的插入位置是随机的，使用中就会形成碎片。所以我们最好自己定义一个自增主键。

# 最佳实践：自增ID做主键
通常我们都会在表中设一个自增ID主键，它可以带来如下好处：
* ID只增不减，所以每写满16K的数据块，就可以开辟新块，不会发生插入数据导致页分裂
* 主键本身也是一个索引，索引就要占据硬盘和内存空间，所以在不影响使用的情况下容量越小越好。默认int型主键，容量足够小。
* 对于长期维护的复杂业务，表中有一个自增ID对于后期进行数据库迁移、数据库结构变更等操作更为方便，主键可以提供顺序、ID标识。

**PS**：<br>
关于自增主键的一个潜在问题，当数值达到最大值后，每一次新增数据都会引起错误。所以自增主键范围要估计好或及时扩容。


