---
layout: post
title:  "Redis数据类型和应用场景"
date:   2018-09-12 10:00:00 +0800
tags: Redis
---

Redis命令参考：[http://redisdoc.com](http://redisdoc.com)

__注意：__ Redis的命令不区分大小写，但是key严格区分大小写

### String（字符串）
* 特性

二进制安全，可以包含任何数据，比如一个jpg图片或者序列化后的对象，value最大512M。

* 场景

网络锁

* 示例

```
redis 127.0.0.1:6379> SET name "value"
OK
redis 127.0.0.1:6379> GET name
"value"
```

### Hash(字典)
* 特性

一个键值(key=>value)对集合，适合存储对象(把对象以类似json的方式序列化)

* 场景

存储、读取、修改用户属性

* 示例

```
redis 127.0.0.1:6379> HMSET myhash field1 "value1" field2 "value2"
"OK"
redis 127.0.0.1:6379> HGET myhash field1
"value1"
```

### List(列表)
* 特性

双向链表，增删快，可操作某一段元素

* 场景

消息队列

* 示例

```
redis 127.0.0.1:6379> LPUSH dqname value1
(integer) 1
redis 127.0.0.1:6379> LPUSH dqname value2
(integer) 2
redis 127.0.0.1:6379> RPUSH dqname value3
(integer) 3
redis 127.0.0.1:6379> LRANGE dqname 0 10
1) "value2"
2) "value1"
3) "value3"
```

### Set(集合)
* 特性

哈希表实现,元素不重复。CRUD操作时间复杂度都为O(1)，提供了求交集、并集、差集等操作

* 场景

共同好友

* 示例

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
* 特性

将Set中的元素增加一个权重参数score,元素按score有序排列。数据插入集合时,已经进行天然排序

* 场景

排行榜、带权重的消息队列

* 示例

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