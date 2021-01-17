---
layout: post
title: "MySQL Lock"
date: 2020-04-19 23:00:00 +0800
tags: MySQL
---

![MySQL Lock](/assets/images/2020-04-19-MySQL_Lock_1.jpg)
锁机制是为了解决并发问题，而关键问题就是保证**事务**。MySQL5.5+版本将默认引擎从 MYISAM 换为 InnoDB 可以高效的支持事务。

# 用锁保证事务

- 示例 1：
  1. A 从银行转账 100 给 B；
  2. 同时 C 从 A 的账户取走 100；

其中 1 是由两个动作组成的：A 账户减少 100、B 账户增加 100。如果 C 的动作夹在中间执行，就可能造成最终错误的结果。

- 示例 2:
  1. 统计当前人数，然后做一些事情，最后按照人数写入一个值。
  2. 在 1 执行过程中，新增一个人。

2 的操作可能引起 1 的结果错误。

## 事务的四大特性 （ACID）

1. 原子性（Atomicity） 事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
2. 一致性（Consistency）事务前后数据的完整性必须保持一致。
3. 隔离性（Isolation）多个用户并发访问数据库时，一个用户的事务不能被其它用户的事务所干扰，多个并发事务之间的数据要相互隔离。
4. 持久性（Durability）一个事务一旦被提交，它对数据库中的数据改变就是永久性的。

## 可能引发的问题

并发执行事务操作，可能造成下面问题：

- **脏读(Dirty Read)**
  事务 A 读到了事务 B **未提交**的数据。
- **不可重复读(Non-repeatable Read)**
  事务 A 第一次查询得到一行记录 row1，事务 B **提交**修改后，事务 A 第二次查询得到 row1，但列内容发生了变化。
  - 不可重复读对应`update`和`delete`操作
- **幻读(Phantom)**
  事务 A 第一次查询得到一行记录 row1，事务 B **提交**修改后，事务 A 第二次查询得到两行记录 row1 和 row2。读到了原本不存在的新记录。
  - 幻读对应`insert`操作
- **丢失更新**
  事务 A B 两次更新顺序到来，最后结果应该是 B，但是在并发更新结束后，结果是 A，B 被覆盖了。

## 四种隔离级别及其特点

| 隔离级别                  | 存在脏读 | 存在不可重复读 | 存在幻读 |
| ------------------------- | -------- | -------------- | -------- |
| Read Uncommited(未提交读) | Y        | Y              | Y        |
| Read Commited(已提交读)   | N        | Y              | Y        |
| Repeatable Read(可重复读) | N        | N              | Y        |
| Serializable(串行化)      | N        | N              | N        |

- 随着从上到下隔离级别越来越严格，同时并发性能越来越差。`Serializable`无法实现并发，所以实际中不使用。
- MySQL 的默认隔离级别是`Repeatable Read(可重复读)`

## 事务隔离级别的实现方式

- **LBCC(Lock-Based Concurrent Control)**
- **MVCC(Multi-Version Concurrent Control)**

- InnoDB 是将上面两种方式结合实现的事务

## MySQL 事务相关基本操作

- AutoCommit(自动提交)
  `show variables like 'autocommit';`这个配置参数是默认打开的，表示每个 SQL 语句自动提交，说明本条 SQL 是符合事务的。
- 手动控制
  在事务开头`begin`、最后`commit`或`rollback`，可以手动控制事务的执行。
- 设置隔离级别
  `set session transaction isolation level repeatable read;`

# 锁的分类

- **LBCC**是 InnoDB 实现事务的主要方式

## 表锁与行锁的区别：

- 锁定粒度：表锁 > 行锁，表锁范围大
- 加锁效率：表锁 > 行锁，表锁加锁快
- 冲突概率：表锁 > 行锁，
- 并发性能：表锁 < 行锁，行锁并发性更好

* 示例：
  预定旅店时，如果一间一间定，比较复杂；整体包旅店，可以快速知道结果；

## 根据操作类型分为：

- **读锁**(共享锁、S 锁(**S**hare Lock))
  - 对同一个数据，多个读操作可以同时进行，互不干扰。
  - 读锁可以先锁定数据，避免数据在事务操作结束前被修改
  - 开启读锁的语句`select * from ... where ... lock in share mode;`
- **写锁**(互斥锁、排他锁、X 锁(E**x**clusive Lock))
  - 如果当前操作没有完毕，则无法进行其他的读/写操作。
  - 开启写锁的语句`select * from ... where ... for update;`
  - `insert into`、`update`、`delete from`都可以触发写锁
- **意向锁**(Intention Lock)
  意向锁是表锁，是自动创建的，不是手动创建的。意向锁用于加**表锁**时直接判断是否有已锁定的情况，避免逐行遍历判断效率低。
  意向锁分为两种：

  - **意向共享锁**(IS 锁(**I**ntention **S**hare Lock))
    事务准备给行加共享锁之前，要先能获取 IS 锁
  - **意向排他锁**(IX 锁(**I**ntention E**x**clusive Lock))
    事务准备给行加排他锁之前，要先能获取 IX 锁

## 根据操作范围分为：

- **表锁**
  对一张表加锁。如 MyISAM 存储引擎使用表锁，开销小、加锁快、无死锁；锁范围大，容易发生冲突，并发度低。
- **行锁**
  对一条数据加锁。如 InnoDB 存储引擎默认使用行锁，开销大、加锁慢、容易发生死锁；锁范围较小，不易发生锁冲突，并发度高。
  - InnoDB 也有表锁
- 页锁

* 操作行锁要通过主键或唯一索引
* 如果没有索引，行锁会转为表锁。例如，在索引失效时，行锁会退化为表锁。

## 根据算法(锁区间)分为：

- **记录锁**(Record Lock)
  - 具体锁定某行，如 1 5 10
  - 等值查询，精准匹配
  - `select * from ... where id=1 for update;`
  - 记录锁要命中主键或唯一索引
- **间隙锁**(Gap Lock)
  - Gap Lock 只存在与 RR(Repeatable Read)隔离级别
  - 锁定具体行之间的开区间，如 (-∞, 1) (5, 10) (10, +∞)，不包含记录本身
  - `select * from ... where id>5 and id<10 for update;`
  - 间隙锁是以当前数据库状态划分间隙的，所以锁定的实际范围可能大于 SQL 语句范围。比如数据库里有两条数据 1 10，那么会自动划为三段间隙：(-∞, 1)、(1, 10)、(10, +∞)。如果执行 SQL`select * from ... where id>15`，这时 id==12 的元素虽然不在`id>15`范围，但是也被锁住了，真正锁住的间隙是(10, +∞)
- **临键锁**(Next-Key Lock)

  - 锁定左开右闭区间，如 (-∞, 1] (5, 10] (10, +∞)，其中闭区间端是一个记录
  - 临键锁的功能是在 RR 级别防止幻读
  - 当间隙锁范围内命中了记录时，就会触发临键锁
  - 假设有数据 1 5 10 15，执行 SQL`select * from .. where id>1 and id<8`。这时由于 5 在间隙锁内被命中，实际锁住的区间是：(1,10]，注意 10 也被锁住了

- 只使用唯一索引查询，并且只锁定一条记录时，innoDB 会使用行锁
- 只使用唯一索引查询，但是检索条件是范围检索，或者是唯一检索然而检索结果不存在（试图锁住不存在的数据）时，会产生 Next-Key Lock。
- 使用普通索引检索时，不管是何种查询，只要加锁，都会产生间隙锁。
- 同时使用唯一索引和普通索引时，由于数据行是优先根据普通索引排序，再根据唯一索引排序，所以也会产生间隙锁。

# MVCC 机制

当输入`begin`时，对数据库进行一个"快照"

- "快照"的实现机制

  1. InnoDB 在每个事务 begin 时生成一个"事务 Id"(transaction_id)，transaction_id 保证严格自增
  2. 每行数据有多个版本，每次更新都会生成一个版本，并且版本号(row_trx_id)与事务 Id(transaction_id) 是对应的
  3. 根据当前的隔离级别，在 select 时读取不同的版本号的数据

- 每个事务有三种相关的 Id:

  - **transaction_id**
    `begin`时由数据库上一个 id 自增产生唯一 id，表示这次事务的唯一标识
  - **row_trx_id**
    一行数据可以同时有多个修改记录，每个记录有一个`row_trx_id`，表示哪个`transaction_id`commit 的数据。
  - **up_limit_id**
    在事务进行中，每条相关记录会有一个`up_limit_id`，表示本条记录应该信哪个`row_trx_id`。`up_limit_id`的默认值是创建事务时，最后一个`transaction_id`(未创建当前事务 id 前)

- 数据的读取方式，有两种：

  - **快照读**
    读取数据时，按照事务起始的`up_limit_id`读数据，就是快照读。
    如果事务操作过程中，读取数据之前没有更新数据，则能够保证都是快照读，保证不会发生**幻读**
  - **当前读**
    如果事务过程中发生过修改，则对应数据的`up_limit_id`会被更新为当`transaction_id`，之后会读取自己最新的修改结果(自己的修改可能是基于别人已提交修改的)，这叫做当前读

- RR 隔离级别下
  - 如果在 begin 后，第一次读取某行值的时候会记录一个快照，之后如果没有更新该字段，则一直保持同一个值。
  - 当中间发生更新动作时，会自动获取结合了别人已提交的最新值(up_limit_id 更新)。
    - 在更新过程中，尽量使用原有变量名进行增、减操作，避免将数据取到变量里再写入，可能引起数据不一致问题。

# 加锁 SQL

## 表锁

- `lock table t1 read/write` 加一个表级的读/写锁
- `unlock tables` 释放锁，也可以用事务解锁
- `show open tables` 查看表的加锁数量
- `show status like '%Table%'` 查看表锁情况，其中`Table_locks_immediate`是立即可加锁的表数量，`Table_locks_waited`是需要等待的表锁数。
  - 通常用`Table_locks_immediate/Table_locks_waited`作为参考数据，如果 > 5000 建议采用 InnoDB，否则用 MyISAM 引擎。

## 行锁

- 通常增删改是通过事务来加锁的
- 可以在查询语句后面加`for update`开启一个行写锁，如:`select * from t1 where id = 1 for update;`
- `show status like '%innodb_row_lock%'` 查看行锁情况(从系统启动到现在)
  - `innodb_row_lock_current_waits` 当前正在等待的锁的数量
  - `innodb_row_lock_time` 等待总时长
  - `innodb_row_lock_time_avg` 平均等待时长
  - `innodb_row_lock_time_max` 最大等待时长
  - `innodb_row_lock_waits` 等待次数

# 锁的效果

## 表锁

- 如果某 session 对 A 表加读锁，则可以对 A 表进行读操作、不可以写操作，当前会话不可以对其他表进行读/写操作。
  - 这时另一个 session 对 A 表可以读，写时会等待，对其他表可以读/写操作。
- 如果某 session 对 A 表加写锁，则可以对 A 表进行任何操作，但是不可以对其他表进行操作。
  - 这时另一个 session 对 A 表任何操作将等待。

# 锁的流程

MyISAM 在执行查询语句(select)之前，会自动给涉及的所有表加读锁；在执行增删改之前，会自动给涉及的表加写锁。

# mysql 8.0 新特性

## NOWAIT 以及

NOWAIT 表示当无法获取到锁时直接返回错误，而不是等待

`select * from ... for update nowait;`

- 在已被别人锁时会立刻返回错误

## SKIP LOCKED

SKIP LOCKED 表示忽略那些已经被其他 session 占有行锁的记录。
