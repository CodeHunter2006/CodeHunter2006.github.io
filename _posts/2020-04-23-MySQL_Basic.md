---
layout: post
title: "MySQL Basic"
date: 2020-04-23 23:00:00 +0800
tags: MySQL
---

## PROCEDURE(存储过程)/FUNCTION(存储函数)

两者都是指提前在 MySQL 服务端编译好可被调用的函数，用来提高查询计算速度。
以前之后 FUNCTION 可以有返回值，后来的版本 PROCEDURE 和 FUNCTION 都可以通过 OUT 参数返回值了，两者差别就不大了。

- 一般只有一个返回值的，用 FUNCTION，否则用 PROCEDURE
- 通常不会用到，除非因为网络开销太大，才会把逻辑做到 MySQL 去做，语法只是普通脚本，并不是很好用

# 常用数据类型

- 三种基本类型：字符串、数字、时间
- char(m)
  固定长度字符串，m 取值范围 1 ～ 255。如果写入的字符串长度不足 m，会自动补齐; 长度超过 m 被截断。由于长度对齐，可以随机跳跃，查询效率高。
- varchar(m)
  可变长度字符串，m 取值范围 1~255，超过截断。不自动补齐，节省空间。
- tinytext
  最多存放 255 字符，很少使用
- text
  最多存放 65535((2^16)-1)个字符
- MEDIUMTEXT
  最大长度 16,777,215
- longtext
  最多存放 4294967292((2^32)-4)个字符
- BOOL/Bit
  都占 1 字节
- tinyint
  - 有符号，-128 ～ 127
  - 无符号，0 ～ 255(2^8)
- smallint
  - 有符号, -32768 ~ 32767
  - 无符号, 0 ~ 65535(2^16)
- int
  - 范围 2^32
- bigint
  - 范围 2^64
- float
  - 精确到小数点后 8 位(2^32)
- double
  - 精确到小数点后 16 位(2^64)
- decimal(m,n)
  货币类型，m 总位数，n 小数位数
- datatime
  混合型，8 字节，年-月-日 时:分:秒，Y-m-d H:i:s
- timestamp
  混合型，4 字节，格式同上
- date
  日期，3 字节，年-月-日, Y-m-d
- time
  时间，3 字节，时:分:秒, H:i:s
- Blob
  最大 64KB
- LongBlob
  最大 4GB

# 数据类型的属性

- NULL
  数据列可包含 NULL 值
- NOT NULL
  数据列不允许包含 NULL 值
- DEFAULT
  默认值
- PRIMARY KEY
  主键
- AUTO_INCREMENT
  自动递增，适用于整数类型
- UNSIGNED
  无符号
- CHARACTER SET name
  指定一个字符集

* int(m)
  m 表示显示时如果不足 m 位，则自动左补 0

# 常量

- `CURRENT_TIMESTAMP`
  当前时间戳
  `xxx timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP`

# 常见问题

- 用`mysql -h xxx -uxxxx -pxxx`连接时，报 1045 错误
  - 原因: 有时密码符号较复杂，在 shell 中直接 `-pxxx` 会被转义为无效符号
  - 解决 1：只输入`-p`，然后被要求输入密码时再输入
  - 解决 2：用转义符`\`提前转义特殊字符，如`$`
