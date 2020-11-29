---
layout: post
title: "Redis Underlying Data Structure"
date: 2020-11-23 22:00:00 +0800
tags: Redis
---

之前总结过 Redis 的 8 种数据类型(String/List/Hash/Set/ZSet/Stream/Geo/HyperLogLog)，而同一种数据类型可能根据数据特性、元素数据量的不同，底层的数据结构也会变化。这里记录一下 Redis 底层的数据结构。

- `TYPE keyName`可以返回数据类型
- `OBJECT ENCODING keyName`可以返回 key 对应的实际数据结构

# SDS(Simple Dynamic String)

Redis 是用 C 语言写的，而 C 语言默认的字符串(以'\0'结尾)的安全性、效率都较低，所以 Redis 中新增了 SDS 作为默认字符串类型。

```C++
struct sdshdr{
     // 记录buf数组中已使用字节的数量
     // 等于 SDS 保存字符串的长度
     int len;
     // 记录 buf 数组中未使用字节的数量
     int free;
     // 字节数组，用于保存字符串
     char buf[];
}
```

- 优点：

  1. 取字符串长度时，时间复杂度 O(1)
     以前 C 语言的字符串要遍历到'\0'时才能取得字符串长度，时间复杂度为 O(n)
  2. 记录了 buf 长度，避免缓冲区溢出
     C 语言字符串执行 strcat 函数时，如果内存不足，可能发生溢出的情况
  3. 利用已有 buf，减少修改字符串时内存重新分配次数
     C 语言字符串是常量，不能修改，只能重新申请内存，虽然减少了重复字符串的内存占用，但是不利于字符串经常修改的情况
     - **预分配**：在字符串长度增加时，可以提前预分配一些内存，长于最初的字符串长度，避免后续频繁申请内存
     - **惰性释放**：在缩短字符串长度时，可以先不释放内存，后续根据 free 字段判断是否释放内存
  4. 二进制安全
     由于 buf 中的元素不限制为字符，所有 Redis 可以存储任何二进制的数据到 SDS 中
  5. 兼容部分 C 字符串函数
     有些情况下，C 语言`<string.h>`库中的一些字符串函数也可以用于 buf 操作

- SDS 除了用于保存字符串外，还可以作为缓冲区 buff 使用。比如 AOF 模块中的 AOF 缓冲区以及客户端状态中的输入缓冲区

# Link List(链表)

# Dictionary(HashMap)

# Skip List(跳跃表)

参考：[算法学习，极限数据结构之——跳跃表(Skip List)](/2019/07/07/Algorithm_Skiplist/)

# Int Set(整数集合)

# Zip List(压缩表)

# Geo Hash(Geography Hash)

参考：[Redis GeoHash Data Structure](/2020/11/28/Redis_GeoHash/)

# Hyper Log Log
