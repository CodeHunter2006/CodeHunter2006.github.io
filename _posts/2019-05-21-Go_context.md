---
layout: post
title:  "Go中context包在并发中的使用"
date:   2019-05-21 10:00:00 +0800
tags: Go HighConcurrency
---

# 网络并发调用
在处理可以并发的事物时，我们经常会起多个goroutine进行处理，通过各自的channel等待结果。当某些结果先到达时，我们要把结果存储下来，等待所有结果都取得后再继续。

![并发调用](/assets/images/2019-05-21-Go_context_1.png)
同样的事物在分布式网络下，我们也会通过goroutine来做。例如用户请求某一信息A，我们需要信息B和C两个信息来综合结果，而B、C分别在两个其他的网络中，所以我们同时发起B、C等待结果后返回。

但是网络存在不稳定因素，例如B可能没有返回结果，于是我们设置了超时并起一个D执行与B相同的任务，然后D顺利返回了。由于某种原因，B后来也返回了，带来了结果，但是由于channel没有人读，B发生了goroutine泄漏。

或者是另一种情况，我们有多个网络可以提供服务，同时启动多个goroutine，如果有谁返回最快则撤销其他goroutine，把最快的结果返回即可。这样在用户看来，很快得到了答案，他不关心后台有多少工作。

在上面的场景中，我们发现了两个并发调用时的需求：
* 每次调用，我们都是想等待一些结果，我们希望有方式可以把中间结果先存储起来
* 我们希望能设置超时、重调的机制来应对网络的不稳定问题
* 当我们获得所有结果后，希望能把之前发起的多余goroutine撤销掉(优雅的关闭)，避免发生泄漏

# context解决方案
![并发调用添加context](/assets/images/2019-05-21-Go_context_2.png)
Go1.7开始，context包被加入了标准库。context正好可以解决我们上面三个痛点。

使用结构如图所示，在每个goroutine中带入一个Context对象，负责返回的数据存储、超时检查、对已经启动的goroutine进行关闭。

Context对象之间会形成一个与goroutine调用结构一致的树形结构。父goroutine自己拥有一个Context对象，在新建子goroutine时会先根据自己的Context对象创建一个子Context,然后传给goroutine的启动函数。子goroutine拿到属于自己的Context对象后可以获取当前的超时、是否被撤销的信息，同时可以将结果写入Context对象保存。

# context包核心接口解析
context包的核心是Context接口：
```
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```
* Deadline()函数返回当前Context对象的超时时间，goroutine可以根据这个时间设置自己的子goroutine的超时时间。如果没有设置，则ok为false。
* Done() 返回一个判断当前Context对象是否被关闭的chan，在子goroutine中可以select这个chan，进行撤销消息的等待。
* Err() 如果当前Context关闭了，可以返回关闭的原因。
* Value() 通过key取得Context对象中存储的对象，这个对象是由父goroutine设定的，子goroutine可以取出来进行赋值操作

# context包中其他函数
```
func Background() Context
```
用于创建根Context对象。其实就是一个空对象，因为根Context对象所属的goroutine没有超时、不能撤销。

```
type CancelFunc func()
```
撤销函数对象，该函数对象用于父goroutine调用Context对象对子goroutine进行撤销操作。

```
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```
用于创建Context对象。传入父Context对象，返回子Context对象和对应的撤销函数对象。

```
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```
同`WithCancel`，增加了超时设定。

```
func WithValue(parent Context, key interface{}, val interface{}) Context
```
同`WithCancel`，增加了key/value值。

```
func TODO() Context
```
有些情况下，我们还没想好怎么设置Context，可以先用这个对象占位。

# 注意事项
虽然Go提供了context包，但是如果没有按照设计的方式使用会引起问题，所以官方文档中给出了下面建议：
* Do not store Contexts inside a struct type; instead, pass a Context explicitly to each function that needs it. The Context should be the first parameter, typically named ctx:
* Do not pass a nil Context, even if a function permits it. Pass context.TODO if you are unsure about which Context to use.
* Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions.
* The same Context may be passed to functions running in different goroutines; Contexts are safe for simultaneous use by multiple goroutines.

# 其他问题
* Context对象是线程安全的，但是Context.Value()取出的对象不是线程安全的，使用的时候要注意。
* Context的使用场景是一次调用的上下文，类似闭包的使用，所以上面说"不要保存为成员变量"。
* 如果把同一个Context传到很多goroutine中，由于Context对象是线程安全的会隐式加锁，所以可能出现所竞争的情况。可以通过创建子Context避免。

# 示例分析
在 [Go Concurrency Patterns: Context - The Go Blog](https://blog.golang.org/context) 中讲述了Google API的一段代码，如何利用Context对象进行超时撤销、数据返回。其中有一些代码调用模式比较典型，在这里分析一下(直接在代码加注释)：
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
这是最初的根goroutine，不可以被撤销。注意最后一句，当下一级Context和CancelFunc同时创建好之后，立即defer CancelFunc的调用，也就是无论最后是超时还是返回了正确值，都会调用CancelFunc。

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


引用：<br/>
[Go Concurrency Patterns: Timing out, moving on - The Go Blog](https://blog.golang.org/go-concurrency-patterns-timing-out-and)<br/>
[Go Concurrency Patterns: Pipelines and cancellation - The Go Blog](https://blog.golang.org/pipelines)<br/>
[Go Concurrency Patterns: Context - The Go Blog](https://blog.golang.org/context)<br/>
[理解GO CONTEXT机制](https://www.cnblogs.com/zhangboyu/p/7456606.html)
