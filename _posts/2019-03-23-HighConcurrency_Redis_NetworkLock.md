---
layout: post
title: "利用Redis实现网络锁"
date: 2019-03-23 10:00:00 +0800
tags: HighConcurrency Redis
---

PS: Redis 官网提出[Redlock 算法](http://www.redis.cn/topics/distlock.html)，并推荐了一个 Go 实现的开源网络锁[redsync](https://github.com/go-redsync/redsync)，实现了所有网络锁要点、接口比较友好，本文可以作为实现思路的参考。

在网络集群中，有时需要多个网络节点争抢一个资源，这时为了保证线程安全，要进行网络锁。

Redis 运行于内存中，响应速度快，并且它提供的一些机制正好可以实现内存锁。

### 上锁流程

- 利用"SET key 随机数 EX 秒数 NX"命令，写入一个随机数
- 如果写入成功，则说明上锁成功，就可以开始利用资源；否则随机等待一段时间再试

### 解锁流程

- 利用 Lua 脚本的原子操作，判断当前 key 值是否是之前写入的随机数，如果是则删除 key；如果不是则直接返回

### 关键点

- 写入时，通过 NX 命令，可以判断是否已经有锁，有锁的话就不能覆盖别人的锁
- 设置超时是为了防止一个请求方获得锁后宕机了，导致整个系统无法解锁。设定了超时，就可以及时开锁
- 写入时随机等待一段时间，是为了避免一次并发之后又一次并发，导致性能下降，可以错峰锁定
- 设置随机数是为了避免误删，如果 A 获得锁、写入 1、然后去做业务，但是 A 超时了，B 获得锁、写入 1,当 A 再来解锁是，看到值是 1，就会误删。设置了随机数，就不会发生这种误删的情况。
- 解锁时要保证原子操作，否则检查与删除中间可能发生变化，导致误删

### 实现代码

```Golang
package main

import (
	"fmt"
	"math/rand"
	"strings"
	"sync"
	"testing"
	"time"

	"github.com/gomodule/redigo/redis"
)

type NetMutex struct {
	targetNet string // 目标网络
	targetKey string // 目标key
	randomNum int64  // 随机数
}

func (p *NetMutex) Lock() {
	conn, err := redis.Dial("tcp", p.targetNet)
	if err != nil {
		panic(err.Error())
	}
	defer conn.Close()

	for isTrying := true; isTrying; {
		res, err := conn.Do("SET", p.targetKey, p.randomNum, "PX", SleepTime*2, "NX")
		//fmt.Printf("%T, %v\n", res, res)
		if err != nil {
			panic(err.Error())
		}
		if res != nil {
			// 获得锁
			isTrying = false
		} else {
			// 没有获得
			time.Sleep(time.Millisecond * time.Duration(rand.Int63n(SleepTime)))
		}
	}
}

func (p *NetMutex) Unlock() {
	conn, err := redis.Dial("tcp", p.targetNet)
	if err != nil {
		panic(err.Error())
	}
	defer conn.Close()

	// 解锁脚本
	unlockScript := new(strings.Builder)
	unlockScript.WriteString("if (redis.call('get',KEYS[1]) == ARGV[1]) ")
	unlockScript.WriteString("then ")
	unlockScript.WriteString("redis.call('DEL',KEYS[1]) ")
	unlockScript.WriteString("return 1 ")
	unlockScript.WriteString("else ")
	unlockScript.WriteString("return 0 ")
	unlockScript.WriteString("end ")
	// 执行脚本
	_, err = redis.Int64(conn.Do("eval", unlockScript.String(),
		1, p.targetKey, p.randomNum))
	if err != nil {
		panic(err.Error())
	}
	//fmt.Println("eval result: ", res)
}

func newNetMutex() (pMtx *NetMutex) {
	pMtx = &NetMutex{"localhost:6379", "ntlock", rand.Int63()}
	return
}

const SleepTime int64 = 100

func TestNetLock(t *testing.T) {
	var mtx sync.Locker = newNetMutex()
	startWG, endWG := sync.WaitGroup{}, sync.WaitGroup{}

	const TotalNum = 100
	count := 0

	startWG.Add(TotalNum)
	endWG.Add(TotalNum)
	for i := 0; i < TotalNum; i++ {
		go func(id int) {
			defer endWG.Done()

			startWG.Wait()

			mtx.Lock()
			defer mtx.Unlock()
			count++
			fmt.Println("I'm ", id)
			time.Sleep(time.Millisecond * time.Duration(SleepTime))
		}(i)
		startWG.Done()
	}

	endWG.Wait()
	fmt.Println("count :", count)
}
```
