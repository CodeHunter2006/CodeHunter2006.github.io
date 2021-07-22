---
layout: post
title: "Go中context包在并发中的使用"
date: 2019-05-21 10:00:00 +0800
tags: Go HighConcurrency
---

# 网络并发调用

在处理可以并发的事物时，我们经常会起多个 goroutine 进行处理，通过各自的 channel 等待结果。当某些结果先到达时，我们要把结果存储下来，等待所有结果都取得后再继续。

![并发调用](/assets/images/2019-05-21-Go_context_1.png)
同样的事物在分布式网络下，我们也会通过 goroutine 来做。例如用户请求某一信息 A，我们需要信息 B 和 C 两个信息来综合结果，而 B、C 分别在两个其他的网络中，所以我们同时发起 B、C 等待结果后返回。

但是网络存在不稳定因素，例如 B 可能没有返回结果，于是我们设置了超时并起一个 D 执行与 B 相同的任务，然后 D 顺利返回了。由于某种原因，B 后来也返回了，带来了结果，但是由于 channel 没有人读，B 发生了 goroutine 泄漏。

或者是另一种情况，我们有多个网络可以提供服务，同时启动多个 goroutine，如果有谁返回最快则撤销其他 goroutine，把最快的结果返回即可。这样在用户看来，很快得到了答案，他不关心后台有多少工作。

在上面的场景中，我们发现了两个并发调用时的需求：

- 每次调用，我们都是想等待一些结果，我们希望有方式可以把中间结果先存储起来
- 我们希望能设置超时、重调的机制来应对网络的不稳定问题
- 当我们获得所有结果后，希望能把之前发起的多余 goroutine 撤销掉(优雅的关闭)，避免发生泄漏

# context 解决方案

![并发调用添加context](/assets/images/2019-05-21-Go_context_2.png)
Go1.7 开始，context 包被加入了标准库。context 正好可以解决我们上面三个痛点。

使用结构如图所示，在每个 goroutine 中带入一个 Context 对象，负责返回的数据存储、超时检查、对已经启动的 goroutine 进行关闭。

Context 对象之间会形成一个与 goroutine 调用结构一致的树形结构。父 goroutine 自己拥有一个 Context 对象，在新建子 goroutine 时会先根据自己的 Context 对象创建一个子 Context,然后传给 goroutine 的启动函数。子 goroutine 拿到属于自己的 Context 对象后可以获取当前的超时、是否被撤销的信息，同时可以将结果写入 Context 对象保存。

# context 包核心接口解析

context 包的核心是 Context 接口：

```
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- Deadline()函数返回当前 Context 对象的超时时间，goroutine 可以根据这个时间设置自己的子 goroutine 的超时时间。如果没有设置，则 ok 为 false。
- Done() 返回一个判断当前 Context 对象是否被关闭的 chan，在子 goroutine 中可以 select 这个 chan，进行撤销消息的等待。
- Err() 如果当前 Context 关闭了，可以返回关闭的原因。
- Value() 通过 key 取得 Context 对象中存储的对象，这个对象是由父 goroutine 设定的，子 goroutine 可以取出来进行赋值操作
  - 一般用自定义类型作为 key 而不用内建类型(如 string)，避免跨包使用时产生 key 冲突
  - 往往用`struct{}`做为实际类型，如`type MyKey struct{}`，好处是节省容量
  - 通常导出的 key 通常为指针或 interface 类型，这样确保全局唯一(interface 的`==`在指针类型下会比较指针的地址)

# context 包中其他函数

```
func Background() Context
```

用于创建根 Context 对象。其实就是一个空对象，因为根 Context 对象所属的 goroutine 没有超时、不能撤销。

```
type CancelFunc func()
```

撤销函数对象，该函数对象用于父 goroutine 调用 Context 对象对子 goroutine 进行撤销操作。

```
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

用于创建 Context 对象。传入父 Context 对象，返回子 Context 对象和对应的撤销函数对象。

```
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

同`WithCancel`，增加了超时设定。

```
func WithValue(parent Context, key interface{}, val interface{}) Context
```

同`WithCancel`，为新增的子对象增加了 key/value 值。

```
func TODO() Context
```

有些情况下，我们还没想好怎么设置 Context，可以先用这个对象占位。

# 注意事项

虽然 Go 提供了 context 包，但是如果没有按照设计的方式使用会引起问题，所以官方文档中给出了下面建议：

- Do not store Contexts inside a struct type; instead, pass a Context explicitly to each function that needs it. The Context should be the first parameter, typically named ctx:
- Do not pass a nil Context, even if a function permits it. Pass context.TODO if you are unsure about which Context to use.
- Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions.
- The same Context may be passed to functions running in different goroutines; Contexts are safe for simultaneous use by multiple goroutines.

# 其他问题

- Context 对象是线程安全的，但是 Context.Value()取出的对象不是线程安全的，使用的时候要注意。
- Context 的使用场景是一次调用的上下文，类似闭包的使用，所以上面说"不要保存为成员变量"。
- 如果把同一个 Context 传到很多 goroutine 中，由于 Context 对象是线程安全的会隐式加锁，所以可能出现所竞争的情况。可以通过创建子 Context 避免。

# 示例分析

在 [Go Concurrency Patterns: Context - The Go Blog](https://blog.golang.org/context) 中讲述了 Google API 的一段代码，如何利用 Context 对象进行超时撤销、数据返回。其中有一些代码调用模式比较典型，在这里分析一下(直接在代码加注释)：

```
func handleSearch(w http.ResponseWriter, req *http.Request) {
    // ctx is the Context for this handler. Calling cancel closes the
    // ctx.Done channel, which is the cancellation signal for requests
    // started by this handler.
    var (
        ctx    context.Context
        cancel context.CancelFunc
    )
    timeout, err := time.ParseDuration(req.FormValue("timeout"))
    if err == nil {
        // The request has a timeout, so create a context that is
        // canceled automatically when the timeout expires.
        ctx, cancel = context.WithTimeout(context.Background(), timeout)
    } else {
        ctx, cancel = context.WithCancel(context.Background())
    }
    defer cancel() // Cancel ctx as soon as handleSearch returns.
```

这是最初的根 goroutine，不可以被撤销。注意最后一句，当下一级 Context 和 CancelFunc 同时创建好之后，立即 defer CancelFunc 的调用，也就是无论最后是超时还是返回了正确值，都会调用 CancelFunc。

```
func httpDo(ctx context.Context, req *http.Request, f func(*http.Response, error) error) error {
    // Run the HTTP request in a goroutine and pass the response to f.
    // 注意，这里为什么设置缓冲区为1，因为不希望子goroutine被阻塞(不读就会阻塞对方)，
    // 只想自己被子goroutine阻塞(对方不写，自己就阻塞)
    c := make(chan error, 1)

    // 将当前Context传递给真正执行http的对象，以便其写入结果，这里并不是为了撤销而传入
    req = req.WithContext(ctx)

    go func() { c <- f(http.DefaultClient.Do(req)) }()
    select {
    // 如果当前Context被撤销，那么当前goroutine仍然要等待下一级调用结果，然后再返回。避免下一级goroutine泄漏。
    // 同时返回的错误也不是子goroutine返回的错误，而是当前Context被撤销的原因。
    case <-ctx.Done():
        <-c // Wait for f to return.
        return ctx.Err()

    // 调用子gotoutine有三种结果：正确、错误、超时。正确的话，Context中的map可以记录结果，这里返回nil；
    // 错误的话，可以通过这里返回错误，并告诉父goroutine不必继续等待；超时的话走上面的case。
    case err := <-c:
        return err
    }
}
```

注意上面的用法，在最后一层 goroutine 中调用了网络操作等待最后的结果，由于没有传入 Context 所以无法立即停止调用。在收到了当前 Context 撤销后，要继续等待结果，然后不做任何操作地返回。这样避免在外部函数内部发生泄漏。

引用：<br/>
[Go Concurrency Patterns: Timing out, moving on - The Go Blog](https://blog.golang.org/go-concurrency-patterns-timing-out-and)<br/>
[Go Concurrency Patterns: Pipelines and cancellation - The Go Blog](https://blog.golang.org/pipelines)<br/>
[Go Concurrency Patterns: Context - The Go Blog](https://blog.golang.org/context)<br/>
[理解 GO CONTEXT 机制](https://www.cnblogs.com/zhangboyu/p/7456606.html)
