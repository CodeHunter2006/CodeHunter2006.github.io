---
layout: post
title: "Redis数据类型和应用场景"
date: 2018-09-12 10:00:00 +0800
tags: Redis
---

Redis 命令参考：[http://redisdoc.com](http://redisdoc.com)

**注意：** Redis 的命令不区分大小写，但是 key 严格区分大小写

### String（字符串）

- 特性

二进制安全，可以包含任何数据，比如一个 jpg 图片或者序列化后的对象，value 最大 512M。

- 场景

相当于单机中的变量

- `SETNX` 可是实现简单的网络锁
- `INCR`/`INCRBY`等可是实现高效的计数
- `TTL`等设置超时的命令，可以进行超时计时
- 在 Key 名中通过相同的前缀可以标识唯一对象，再通过后缀标识属性名，可以存储对象及其组合关系成员

* 示例

```
redis 127.0.0.1:6379> SET name "value"
OK
redis 127.0.0.1:6379> GET name
"value"
```

### Bitmap (位图)

- 特性

本质就是很大的(最大 512MB) byte 数组，可以操纵指定位置的 bit 值

- 场景

  - 用来实现 BloomFilter
  - 用位来记录用户在线状态，可以表示 4294967296(`512*1024*1024*8`) 个用户，也就是统计全世界的人
  - 利用位快速排序，可以给数字范围在 4294967296 内的数组排序(元素不重复)
    - 从左到右每一位表示一个数字是否存在
    - 遍历数组，通过移位操作，将数字对应的位置置 1
    - 从左到右遍历每一位，移位换算成原数字，就是已排好序数组
    - 时间复杂度 O(n)
  - 去重，将 Bitmap 看做是 元素 ID 的 set，用于去重

- 示例

```
redis> SETBIT mykey 7 1
(integer) 0
redis> GETBIT mykey 0
(integer) 0
redis> GETBIT mykey 7
(integer) 1
```

### Hash(字典)

- 特性

一个键值(key=>value)对集合，适合存储对象(把对象以类似 json 的方式序列化)

- 场景

相当于单机中的 HashMap

存储、读取、修改特对对象的属性，用 Key 值作为对象标识，用 Field 作为字段名

- 示例

```
redis 127.0.0.1:6379> HMSET myhash field1 "value1" field2 "value2"
"OK"
redis 127.0.0.1:6379> HGET myhash field1
"value1"
```

### List(列表)

- 特性

双向链表，增删快，可操作某一段元素

- 场景

相当于单机中的 vector，可模拟 Queue、Stack

可以作为消息队列

- 示例

```
redis 127.0.0.1:6379> LPUSH dqname value1
(integer) 1
redis 127.0.0.1:6379> LPUSH dqname value2
(integer) 2
redis 127.0.0.1:6379> RPUSH dqname value3
(integer) 3
redis 127.0.0.1:6379> LRANGE dqname 0 -1
1) "value2"
2) "value1"
3) "value3"
```

注意，下标从 0 算起，-1 表示最后一个元素、-2 表示倒数第二个元素...

### Set(集合)

- 特性

哈希表实现,元素不重复。CRUD 操作时间复杂度都为 O(1)，提供了求交集、并集、差集等操作

- 场景

相当于单机中的 HashSet

- 共同好友

- 示例

```
redis 127.0.0.1:6379> SADD setname value1
(integer) 1
redis 127.0.0.1:6379> SADD setname value2
(integer) 1
redis 127.0.0.1:6379> SADD setname value1
(integer) 0
redis 127.0.0.1:6379> SMEMBERS setname
1) "value1"
2) "value2"
```

### Sorted Set(有序集合)

- 特性

将 Set 中的元素增加一个权重参数 score,元素按 score 有序排列。数据插入集合时,已经进行天然排序

- 场景

相当于单机中的 Map(有序 Map)

排行榜、带权重的消息队列

- 示例

```
redis 127.0.0.1:6379> ZADD setname 0 value2
(integer) 1
redis 127.0.0.1:6379> ZADD setname 0 value1
(integer) 1
redis 127.0.0.1:6379> ZADD setname 0 value1
(integer) 0
redis 127.0.0.1:6379> > ZRANGEBYSCORE setname 0 1000
1) "value1"
2) "value2"
```

### Stream

Redis5.0 中新增的数据结构，类似 kafka 的功能。

其实 Stream 的底层实现还是用 List，每个元素有一个唯一 msgid，并保证 msgid 严格递增。
在 Stream 当中，消息是默认持久化的，即便是 Redis 重启，也能够读取到消息。
不同的消费者可以属于一个 Group，对于同一个 Topic(Key)，每个 Group 都会维护一个 Idx 下标，每次消费向后偏移。

- 场景

消息队列

- 示例

`XGROUP CREATE`
创建消费组

`XADD`
添加消息到末尾

### Geo

用于存储地理位置信息

- 示例

`GEOADD Sicily 13.361389 38.115556 "Palermo"`
添加一条地理位置信息，key、经度、纬度、位置名称

`geodist`
计算两个位置之间的距离

`georadius`
返回指定坐标半径范围内的位置元素集合

### HyperLogLog

用来做基数(不重复元素数)统计

- 特性

每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。所需容量不像 Set 随元素数量线性增长。

HyperLogLog 利用概率算法实现。

- 场景

  - 统计注册 IP 数
  - 统计每日访问 IP 数
  - 统计页面实时 UV 数
  - 统计在线用户数
  - 统计用户每天搜索不同词条的个数
  - 统计车辆牌照、人脸等可以用 CV 计算出特征的样本数

- 示例

`PFADD runoobkey "mongodb"`
添加一个元素

`PFCOUNT runoobkey`
返回基数值
