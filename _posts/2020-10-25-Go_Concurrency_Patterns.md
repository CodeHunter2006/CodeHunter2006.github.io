---
layout: post
title: "Go Concurrency Patterns"
date: 2020-10-25 22:20:00 +0800
tags: Go DesignPattern
---

`Do not communicate by sharing memory; instead, share memory by communicating.`
并发编程的问题点总是和共享变量的方式有着重要而微妙的关系，Go+Pipeline 模式可以很好的处理并发设计。
本文参考下面文章，总结 Go 中的一些 goroutine 和 channel 的使用模式。

- [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)
- [Effective Go](https://docs.studygolang.com/doc/effective_go.html)

## Pipeline and Filter 模式关键概念

将输入数据经过一系列处理，输出最终结果，就像是污水经过一系列管道最后流出自来水。其中处理过程的基础组件是过滤器(Filter 也叫 stage)，可以通过各种 stage 的顺序组合变换出不同的结果。通常用于可以并行执行的计算过程。

- **stage/filter** 每一节"水管"就是一个 stage，是基本组件单元，对输入进行一个运算处理
- **pipeline** 某种 stage 的组合形成一个流水线，实现特定功能
- **inbound** 入口，上游的数据进入 stage 时从入口取出
- **outbound** 出口，流向下游的数据放入出口
- **source/producer** 最上游的数据来源
- **sink/consumer** 最下游的数据消费者

在 Go 中，由于 goroutine 和 channel 运行效率高、资源占用小，也可以方便的利用 Pipeline 进行程序设计。

## 基本示例

由三个 stage 组成的 pipeline，实现"取平方"功能

## Fan-out/Fan-in

利用 goroutine 和 channel 可以实现数据的扇入、扇出。

```Go
// fan-in merge
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

## Explicit cancellation

有些情况下不需要读取所有的 inbound，可以提前获得结果并结束处理，比如中间发生了错误、已经计算出了正确值等，这种情况下可以增加一个 channel 作为结束信号。

```Go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}
```

## Bounded parallelism

在一个三个 stage 组成的 pipeline 中，可以通过控制中间处理 stage 的实例数，来限制内存资源占用边界。

```Go
	// Start a fixed number of goroutines to read and digest files.
	c := make(chan result)
	var wg sync.WaitGroup
	const numDigesters = 20
	wg.Add(numDigesters)
	for i := 0; i < numDigesters; i++ {
		go func() {
			digester(done, paths, c)
			wg.Done()
		}()
	}
	go func() {
		wg.Wait()
		close(c)
	}()
```

## A leaky buffer

一个简单的漏桶式缓冲区。

```Go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}

func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}s
```
