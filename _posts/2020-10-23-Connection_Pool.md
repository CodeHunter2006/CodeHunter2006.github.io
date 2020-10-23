---
layout: post
title: "Connection Pool 连接池"
date: 2020-10-23 22:20:00 +0800
tags: Server Golang Pool
---

![连接池](/assets/images/2020-10-23-Connection_Pool_1.png)
服务和服务之间建立网络连接是开发中常见的操作，但是随着连接的次数、长连接的数量增加会产生各种资源的瓶颈，连接池就应运而生了。这里以 Go 的`sql.DB`为例，分析一下连接池注意点。

# 服务连接产生的问题

- 每个连接建立要三次握手、四次挥手，花费时间(在局域网，通常要 3ms 左右建立连接)
- 每个连接，要在连接两端内存中申请 socket buffer，占用内存
- 每个连接关闭时，要释放相关内存，带来 CPU 消耗
- Server 端(被调用端)如果被很多 Client 端连接，每个 Client 端连接不受限制，则可能影响 Server 端的整体性能

# 连接池的功能

- 连接池是主动调用端建立的，维护多个与被动调用放的连接(比如 MySQL 数据库操作，我们的 Server 就是主动端 Server，MySQL 是被动端 Server)
- 连接池维护特定数量的连接并保持长连接，这样随时可用，在大并发请求时减少了建立 MySQL 连接的耗时
- 连接池里的连接在使用时取走、使用后放回，这样可以暂时关闭连接，避免不断释放带来损耗
- 连接池里的连接数是在一定范围内固定的，这样可以保护被动端 Server，不至于因为访问请求过多而导致整体处理效率下降

# 连接池有哪些属性

下面是`sql.DB`代码

```Golang
type DB struct {
	// Atomic access only. At top of struct to prevent mis-alignment
	// on 32-bit platforms. Of type time.Duration.
	waitDuration int64 // Total time waited for new connections.

	connector driver.Connector
	numClosed uint64

	mu           sync.Mutex // protects following fields
	freeConn     []*driverConn
	connRequests map[uint64]chan connRequest
	nextRequest  uint64 // Next key to use in connRequests.
	numOpen      int    // number of opened and pending open connections
	openerCh          chan struct{}
	closed            bool
	dep               map[finalCloser]depSet
	lastPut           map[*driverConn]string // stacktrace of last conn's put; debug only
	maxIdleCount      int                    // zero means defaultMaxIdleConns; negative means 0
	maxOpen           int                    // <= 0 means unlimited
	maxLifetime       time.Duration          // maximum amount of time a connection may be reused
	maxIdleTime       time.Duration          // maximum amount of time a connection may be idle before being closed
	cleanerCh         chan struct{}
	waitCount         int64 // Total number of connections waited for.
	maxIdleClosed     int64 // Total number of connections closed due to idle count.
	maxIdleTimeClosed int64 // Total number of connections closed due to idle time.
	maxLifetimeClosed int64 // Total number of connections closed due to max connection lifetime limit.

	stop func() // stop cancels the connection opener and the session resetter.
}

type Conn struct {
	db *DB

	closemu sync.RWMutex
	dc *driverConn

	done int32
}
```

- `func (db *DB) SetMaxOpenConns(n int)`
  最大连接数。当请求不断增加时，不断创建连接，当连接数达到最大连接数后，新的获取连接请求将被阻塞。
- `func (db *DB) SetMaxIdleConns(n int)`
  最大空闲连接数。当没有连接需求时，连接池会逐步释放连接，直到维持一个空闲连接数，避免突然大并发造成来不及建立连接的情况。
- `func (db *DB) SetConnMaxLifetime(d time.Duration)`
  一个连接的最大使用时长。如果超过这个时长，连接会被关闭。如果关闭后空闲连接数不足，则会新建立一个连接。
- `func (db *DB) SetConnMaxIdleTime(d time.Duration)`
  超出最大空闲连接数的连接，经过多久会被关闭。

- 获取连接流程：
  1. 如果有空闲连接，则获取连接
  2. 没有空闲连接，则建立新连接
  3. 达到最大连接数，则阻塞，直到获得连接
  4. 如果达到设定的时间仍未获得连接，则超时返回 error

# 连接池的一些设计点

- 连接池的连接，通常使用之后要放回池中。如果发生网络错误，则不放回，而直接关闭，这样连接池会创建新连接。
- 每个连接要记录一个最新有效时间，超过了心跳间隔要自动 ping 一下以保持连接的健康状态，下面三点表示有效：
  1. 连接刚刚建立
  2. 连接刚正常使用过，返回的结果正确，没有网络异常
  3. 连接刚经过 ping 操作，收到了 pang
- `DB.openerCh`表示要打开的新连接的队列，使用 channel 可以减少锁的使用
- `DB.cleanerCh`表示要清理的连接队列

# 连接池问题/案例

## 连接池空闲数过小

由于连接池空闲数过小，基本相当于没有。在发生大并发时，资源急剧消耗，造成处理速度慢。通常要测试一个合理的连接数范围，设定到 idel 和 max。

## 同一个连接偶现返回错误结果

- 问题现象：
  偶现通过连接发送的某个请求，返回的结果的内容无法解析出来，发生异常
- 原因分析：
  有 A、B 两个携程，A 携程发出 RequestA 收到 ResponseA。B 携程发出 RequestB 但是不关心 ResponseB。某个连接先被 B 携程获取，发出 RequestB，然后把连接放入连接池，该连接被 A 取得，发出 RequestA，之后立即收到了 ResponseB，由于 A 携程解析 ResponseB，发生异常。
- 解决方案：
  确保每次使用连接时，一定要完成 Request、Response 全过程后才放回连接
