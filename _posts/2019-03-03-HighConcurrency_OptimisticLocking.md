---
layout: post
title:  "秒杀架构与乐观锁"
date:   2019-03-03 10:30:00 +0800
tags: Go HighConcurrency
---
### 秒杀与高并发
"秒杀"是指一种购物模式，在特定时间所有买家同时抢购商品，最后只有与库存数量对应的买家可以买到。

秒杀是一种典型的高并发情况，同一瞬间服务器将承受每秒上百万次的请求，这对正系统架构的高可靠、高可用、高性能要求达到极致。

### 秒杀业务关键点分析
秒杀业务逻辑的服务端基本特征：
* 特定时间集中购买，集中发出请求
* 浏览的人多买的人少，读多写少
* 库存有限只有少数人可以买到，需要加锁竞争

分析了上面的特征后发现，并不是所有的秒杀都能达到上面的特征，只有像双十一这样的秒杀才能达到那么高的并发。

类似的场景还有：
* 过年抢车票
* 大群里抢红包
* 医院资源的在线预约
* 点赞
* ...

秒杀购物的流程是：
* 商品查询
* 创建订单
	* 加入购物车
	* 确认订单
	* 修改库存
	* 等待支付
* 订单支付
* 卖家发货

其中，最关键的时间点是修改库存。因为库存数量是固定的，不可以卖多、卖重，必须保证原子操作，所以这也是秒杀架构实现的关键点。(秒杀架构还有其他关键点，但是这里只讨论库存修改时的问题)

### 乐观锁与悲观锁
为了实现库存的原子操作，多个用户请求之间就要进行锁操作。

所谓悲观锁，就是我们通常使用的锁Mutex，在进行资源操作前提前将资源锁住，然后进行操作，完毕后再释放锁。悲观锁适合 __写多读少__ 的情况，

乐观锁是指锁定前认为进行最终修改的人会很少而读的人多，所以只有在写的瞬间才进行校验，如果校验正确则同时写入、如果校验失败则等待一段时间再尝试，直到得到最终结果(买到了或者卖完了)。乐观锁适合 __读多写少__ 的情况，正好与秒杀逻辑一致。因为读多写少，可以避免实际的锁定，总的吞吐率较高。

乐观锁的这种检查同时写入的机制叫作CAS（Compare and Swap），被广泛应用在cache机制中。

乐观锁有几个关键点：
* 需要某个版本标识，进行校验比对。
在实际使用中，可以通过数据库版本号字段、数据库原库存值作为校验值，判断是否字段被修改过。
* 整个CAS过程要保证原子操作
在Redis数据库中，可以通过Lua脚本进行原子操作。因为Redis是单线程的，所以在执行脚本时不会发生线程不安全问题。使用Redis的WATCH命令也可以实现CAS操作，这里选择Lua实现方案。
* 写入失败后，要等待随机时间后再重试
如果等待相同时间，那么又会同一时间进行抢夺操作，造成不必要的踩踏
* 硬件介质本身的速度问题
利用MySQL也可以实现乐观锁，但是往往还是使用Redis这种缓存数据库，因为硬盘数据库读写每秒速度有限(高速硬盘300~SSD固态硬盘700)，而内存可以达到万级的访问。

### 用Lua+Redis实现乐观锁(goroutine测试)
```
var mtx = new(sync.Mutex)

var optLock = false // true 乐观锁， false 悲观锁

func buy(starWG, endWG *sync.WaitGroup) {
	defer endWG.Done()

	conn, err := redis.Dial("tcp", "localhost:6379")
	if err != nil {
		panic(err.Error())
	}
	defer conn.Close()
	starWG.Wait()

	if optLock {
	} else {
		mtx.Lock()
		defer mtx.Unlock()
	}

OPT_TRY:

	// 获得商品数量
	res, err := redis.Int64(conn.Do("GET", "num"))
	if err != nil {
		panic(err.Error())
	}
	if res > 0 {
		// 购买商品
		if optLock {
			// 编写抢购脚本
			tryBuyScript := new(strings.Builder)
			tryBuyScript.WriteString("if (redis.call('get',KEYS[1]) == ARGV[1]) ")
			tryBuyScript.WriteString("then ")
			tryBuyScript.WriteString("redis.call('DECR',KEYS[1]) ")
			tryBuyScript.WriteString("return 1 ")
			tryBuyScript.WriteString("else ")
			tryBuyScript.WriteString("return 0 ")
			tryBuyScript.WriteString("end ")

			// 执行抢购脚本
			res1, err := redis.Int64(conn.Do("eval", tryBuyScript.String(), 
			1, "num", res))
			if err != nil {
				panic(err.Error())
			}
			//fmt.Println("eval result: ", res)
			if res1 == 1 {
				// 买到
			} else {
				// 没买到
				time.Sleep(time.Millisecond * time.Duration(rand.Int31n(10)))

				goto OPT_TRY
			}
		} else {
			_, err = conn.Do("DECR", "num")
			if err != nil {
				panic(err.Error())
			}
			//fmt.Println("买到啦！")
		}
	} else {
		//fmt.Println("没买到...")
	}
}

func timeoutCheck(tag string, start time.Time) {
	dis := time.Since(start).Seconds() * 1000
	fmt.Println(tag, dis, "millionseconds")
}

func TestSeckill(t *testing.T) {
	const TotalNum, PersonNum int = 10, 500 // 商品总数，买家人数
	t.Log("秒杀测试开始")

	// 写入商品数量
	conn, err := redis.Dial("tcp", "localhost:6379")
	if err != nil {
		panic(err.Error())
	}
	defer conn.Close()
	_, err = conn.Do("SET", "num", TotalNum)
	if err != nil {
		panic(err.Error())
	}

	// 并发开始和结束同步
	startWG, endWG := new(sync.WaitGroup), new(sync.WaitGroup)
	startWG.Add(PersonNum)
	endWG.Add(PersonNum)

	for i := 0; i < PersonNum; i++ {
		go buy(startWG, endWG)
		startWG.Done()
	}
	startTime := time.Now()

	endWG.Wait()
	timeoutCheck("time count", startTime)
	res, err := redis.String(conn.Do("GET", "num"))
	if err != nil {
		panic(err.Error())
	}
	t.Log("剩余数量：" + res)
	_, err = conn.Do("DEL", "num")
	if err != nil {
		panic(err.Error())
	}
	t.Log("秒杀测试结束")
}
```

### 测试结果
在本机上，乐观锁、悲观锁差别很小。两者的差异在网络环境下应该能看出来。


