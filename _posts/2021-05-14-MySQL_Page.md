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
区的下一级是**页**，默认为**16KB**，正好和磁盘一次读取的缓冲区容量一致。一个区最多可以包含 64 页。
页的下一级是**行**，一页可以包含`2~n`行，每行容量由表结构设计时定义。
InnoDB 规定一页至少容纳两行，所以行大小限制为**8000B**。

![B+ Node](/assets/images/2021-05-14-MySQL_Page_3.png)

# 常见问题和应对方法
