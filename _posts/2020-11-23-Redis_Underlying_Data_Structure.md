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

```C
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

Redis 中实现了双向链表，数据结构：

```C
typedef struct listNode{
       // 前置节点
       struct listNode *prev;
       // 后置节点
       struct listNode *next;
       // 节点的值
       void *value;
}listNode

typedef struct list{
     // 表头节点
     listNode *head;
     // 表尾节点
     listNode *tail;
     // 链表所包含的节点数量
     unsigned long len;
     // 节点值复制函数
     void (*free) (void *ptr);
     // 节点值释放函数
     void (*free) (void *ptr);
     // 节点值对比函数
     int (*match) (void *ptr,void *key);
}list;
```

- Redis 链表特性：
  1. 双端：链表具有前置节点和后置节点的引用，获取这两个节点时间复杂度都为 O(1)。
  2. 无环：表头节点的 prev 指针和表尾节点的 next 指针都指向 NULL,对链表的访问都是以 NULL 结束。
  3. 带链表长度计数器：通过 len 属性获取链表长度的时间复杂度为 O(1)。
  4. 多态：链表节点使用`void*`指针来保存节点值，可以保存各种不同类型的值。

# Dictionary(HashTable)

Redis 中实现了哈希表，数据结构：

```C
typedef struct dictht{
     // 哈希表数组
     dictEntry **table;
     // 哈希表大小
     unsigned long size;
     // 哈希表大小掩码，用于计算索引值
     // 总是等于 size-1
     unsigned long sizemask;
     // 该哈希表已有节点的数量
     unsigned long used;
}dictht

typedef struct dictEntry{
     // 键
     void *key;
     // 值
     union{
          void *val;
          uint64_tu64;
          int64_ts64;
     }v;

     // 指向下一个哈希表节点，形成链表
     struct dictEntry *next;
}dictEntry
```

- Redis HashTable 特性：
  1. 哈希算法：
     1. 使用字典设置的哈希函数，计算键 key 的哈希值`hash = dict->type->hashFunction(key);`
     2. 使用哈希表的 sizemask 属性和第一步得到的哈希值，计算索引值`index = hash & dict->ht[x].sizemask;`
  2. 解决哈希冲突："开链法"，同一个 Hash 值的元素用以链表形式连接
  3. 扩容和收缩：
     当哈希表保存的键值对太多或者太少时，就要通过 rerehash(重新散列）来对哈希表进行相应的扩展或者收缩。具体步骤：
     1. 如果执行扩展操作，会基于原哈希表创建一个大小等于 `ht[0].used*2n` 的哈希表（也就是每次扩展都是根据原哈希表已使用的空间扩大一倍创建另一个哈希表）。相反如果执行的是收缩操作，每次收缩是根据已使用空间缩小一倍创建一个新的哈希表。
     2. 重新利用上面的哈希算法，计算索引值，然后将键值对放到新的哈希表位置上。
     3. 所有键值对都迁徙完毕后，释放原哈希表的内存空间。
  4. 触发扩容的条件:(负载因子 = 哈希表已保存节点数量 / 哈希表大小)
     1. 服务器目前没有执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且负载因子大于等于 1。
     2. 服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且负载因子大于等于 5。
  5. 渐近式 rehash
     扩容和收缩操作不是一次性、集中式完成的，而是分多次、渐进式完成的。如果保存在 Redis 中的键值对只有几个几十个，那么 rehash 操作可以瞬间完成，但是如果键值对有几百万，几千万甚至几亿，那么要一次性的进行 rehash，势必会造成 Redis 一段时间内不能进行别的操作。所以 Redis 采用渐进式 rehash,这样在进行渐进式 rehash 期间，字典的删除查找更新等操作可能会在两个哈希表上进行，第一个哈希表没有找到，就会去第二个哈希表上进行查找。但是进行 增加操作，一定是在新的哈希表上进行的。

# Skip List(跳跃表)

参考：[算法学习，极限数据结构之——跳跃表(Skip List)](/2019/07/07/Algorithm_Skiplist/)

# Int Set(整数集合)

整数集合（intset）是 Redis 用于保存整数值的集合抽象数据类型，它可以保存类型为 int16_t、int32_t 或者 int64_t 的整数值，并且保证集合中不会出现重复元素。

```C
typedef struct intset{
     // 编码方式
     uint32_t encoding;
     // 集合包含的元素数量
     uint32_t length;
     // 保存元素的数组
     int8_t contents[];
}intset;
```

- 每个元素都是 contents 数组的一个数据项，它们按照从小到大的顺序排列，并且不包含任何重复项
- length 属性记录了 contents 数组的大小
- 虽然 contents 数组声明为 int8_t 类型，但是实际上 contents 数组并不保存任何 int8_t 类型的值，其真正类型由 encoding 来决定

- 升级
  当我们新增的元素类型比原集合元素类型的长度要大时，需要对整数集合进行升级
  1. 根据新元素类型，扩展整数集合底层数组的大小，并为新元素分配空间
  2. 将底层数组现有的所有元素都转成与新元素相同类型的元素，并将转换后的元素放到正确的位置，放置过程中，维持整个元素顺序都是有序的
  3. 将新元素添加到整数集合中，添加时保持有序

* 降级
  整数集合不支持降级操作，一旦对数组进行了升级，编码就会一直保持升级后的状态

# Zip List(压缩表)

压缩列表是 Redis 为了节省内存而设计的数据结构，在 list、set、zset 容量较小时使用。

- 特点

  - 连续内存，节省空间
  - 每个元素类型可以不同
  - 元素之间有序排列
  - 双向链式结构，可以前、后遍历

- 压缩列表的结构
  `zlbytes, zltail, zllen, entry1, ..., entryN, zlend`

| 属性    | 类型     | 长度 | 用途                                                                                                                                                                                            |
| ------- | -------- | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| zlbytes | uint32_t | 4B   | 记录整个压缩列表占用的内存字节数,<br>在对压缩列表进行内存重分配或计算 zlend 的位置时使用                                                                                                        |
| zltail  | uint32_t | 4B   | 记录压缩列表表尾节点距离压缩列表的起始地址有多少字节,<br>通过这个偏移量，程序无须遍历整个压缩列表就可以确定表尾节点的地址                                                                       |
| zllen   | uint16_t | 2B   | 记录了压缩列表包含的节点数量,<br>当这个属性的值小于 UINT16_MAX(65535)时，这个属性的值就是压缩列表包含节点的数量；<br>当这个值等于 UINT16_MAX 时，节点的真实数量需要遍历整个压缩列表才能计算得出 |
| entryX  | 列表节点 | 不定 | 压缩列表包含的各个节点，节点的长度由节点保存的内容决定                                                                                                                                          |
| zlend   | uint8_t  | 1B   | 特殊值 0xFF(十进制 255)，用于标记压缩列表的末端                                                                                                                                                 |

- entry 结构
  `previous_entry_length, encoding, content`

| 属性                  | 长度   | 用途                                                                                                                                                |
| --------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| previous_entry_length | 1/5B   | 记录前一个数据的长度，以便从后向前遍历。<br>如果前一元素长度小于 254 字节则用 1 字节表示，否则用 5 字节表示(第一字节值为 254，后面四字节存储实际值) |
| encoding              | 1/2/5B | 描述存储内容的编码式，以及长度                                                                                                                      |
| content               | 不定   | 存储的内容                                                                                                                                          |

- 从上面可以看出，Redis 为了节省内存，已做到极致
- 向中间插入元素时，如果 previous_entry_length 本身的长度发生变化，会引发连锁更新，时间复杂度为 O(n)，但好在元素不多，所以不会造成性能问题

# Geo Hash(Geography Hash)

参考：[Redis GeoHash Data Structure](/2020/11/28/Redis_GeoHash/)

# Hyper Log Log

参考：[Redis HyperLogLog Data Structure](/2020/12/03/Redis_HyperLogLog/)
