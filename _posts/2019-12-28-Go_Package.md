---
layout: post
title: "Go Package Notes"
date: 2019-12-28 22:00:00 +0800
tags: Go
---

Go 常用包的功能，及注意点

# 官方

## context

为多携程调用提供三种功能：传值、超时、取消。

- 多个 context 对象可形成树形关联，当父 context 取消后，子 context 也会随之取消；子 context 取消后，父 context 不受影响
- context 是线程安全的

## container/heap

提供(**最小**)堆相关函数和接口，可以延伸实现优先级队列功能。需要自己实现接口，以满足一个可排序的容器，然后结合函数使用。

- 最小元素的下标是 0，可以用来做`peak`处理

```Go
type Interface interface {
    sort.Interface
    Push(x interface{}) // add x as element Len()
    Pop() interface{}   // remove and return element Len() - 1.
}
```

- **注意**
  对 Heap 的操作一定要通过`heap.XXX`函数执行

- Heap 提供了`remove`函数，需要数组下标作为参数，可以通过堆查找实现`O(logn)`删除指定元素

- 对于`[]int`类型的 heap，可以直接内嵌`sort.IntSlice`类型自动实现`sort.Interface`

## container/list

提供链表数据结构

```Go
// 可以利用 Remove + Front/Back 实现 Pop 功能
l.Remove(l.Front()).(int)
```

### WithCancel

cancel 函数可以调用多次，不会发生 panic，只有第一次调用起作用

## encoding/json

对 struct 进行 marshal 和 unmarshal 操作，实现原理利用了反射，所以性能较差，在高并发下可用其他库替代。

- struct 内嵌 interface 只能 marshal 无法 unmarshal，由于字符串无法反向推导类型
- 对 interface 可以进行 marshal，会根据 interface 中保存的实际结构体类型进行 marshal

- unmarshal 时，可以传入几种可识别的类型，比如 struct、slice、map，根据不同的类型结果不同。
  - struct 将按照其中的导出成员变量进行解析
  - slice 将被解析为多个字符串，每个字符串是一个 json 对象
  - map 将被解析为`map[string]interface{}`结构，其中`interface{}`可能是一个 map
  - 也可以直接传入多维 map 结构，如`map[string]map[string]string`，可以自动解析

在 unmarshal 时，传入的参数必须是对应类型的指针，并且保证对象已构造。但是 slice 或 map 的 make 并不是必须的，只要对象自动创建了，不需要 make 初始化。

```Go
var out map[string]interface{}
err = json.Unmarshal(str, &out)
if err != nil {
	log.Panicln(err)
}
```

- unmarshal 时，如果传入了`interface`(无论是`interface{}`还是特定类型的`interface`)，
  如果 interface 已经指向了有效对象，则会 unarshal 成功；如果指向 nil，则会 panic。

- json `[1, "2"]` 用 `[]interface{}` 类型解析时会把`1`解析为`float64`类型，底层思路是尽量把信息保留下来避免丢失。

- 有时 struct 中成员类型是 int64，一个 uint64 的数值可能报错 "json: cannot unmarshal number xxxxxx into Go struct field"
  - Go 中没有 uint64，所以会报错
  - 这种情况下，可以把类型改为 float64，一方面可以接受数值不会报错；另一方面这个数字往往是溢出后的情况，不需要太精确。

```Go
struct Test {
  ignored int64 `json:"-"`  // 忽略该字段
}
```

- 可以通过重写`MarshalJSON/UnmarshalJSON`方法实现自定义的 json 序列化和反序列化

```Go
func (o TestType) MarshalJSON() ([]byte, error) {
  return []byte(fmt.Sprintf("%d", o)), nil
}

func (o *TestType) UnmarshalJSON(data []byte) error {
  var i int64
  err := json.Unmarshal(data, &i)
  if err != nil {
    return err
  }
  *o = TestType(i)
  return nil
}
```

## errors

```Go
import "github.com/pkg/errors"  // 这个包继承了内置的`errors`，提供了更好的功能

err1 := errors.New("this is an error")  // 创建 error 对象
err2 := errors.WithMessage(err1, "upper level error") // wrap with message

fmt.Println(err)  // output all error message
// upper level error: this is an error

fmt.Printf("%+v", err) // output call all message and stacks
/*
upper level error: this is an error
main.main
    /usr/three/main.go:11
main.main
    /usr/three/main.go:15
*/

```

## errorgroup

errorgroup 综合了 waitGroup 和 context 的能力，可以同时等待多个协程、返回错误、协程 cancel

- 优点：
  - 结合了`sync.WaitGroup`，直接`group.Go(...)`启动携程，无需`wg.Add(1)`
  - 如果绑定了 context，任意函数返回 error 会触发 cancel；如果不绑定 context 可以当成普通的 WaitGroup 使用
  - `group.Wait()`可以自动收集第一个 error

```Go
func testErrorGroup() {
	rand.Seed(time.Now().Unix())
	const threadNum = 5
	ctx, cancel := context.WithCancel(context.Background())
	group, errCtx := errgroup.WithContext(ctx)

	for i := 0; i < threadNum; i++ {
		index := i // 避免闭包中引用同一个 i

		group.Go(func() error {
			if index < threadNum-1 {
				for j := 0; j < 10; j++ {
					log.Printf("%v is running %v", index, j)
					time.Sleep(time.Microsecond*time.Duration(rand.Int()%1000) + time.Second)

					select {
					case <-errCtx.Done(): // 检测到已 cancel 后退出
						log.Printf("%v return with error: %v", index, errCtx.Err())
						return fmt.Errorf("%v error: %v", index, errCtx.Err())
					default:
					}
				}
				log.Printf("%v end", index)
			} else {
				time.Sleep(time.Microsecond*time.Duration(rand.Int()%1000) + time.Second*3)
				// 返回 error 时，由于绑定了 context，会自动触发 cancel
				return fmt.Errorf("%v cancel error", index)
			}
			return nil
		})
	}

	err := group.Wait()
	if err == nil {
		log.Println("all done!")
	} else {
		log.Printf("found error: %v", err)  // 输出："found error: 4 cancel error"
	}
}
```

## flag

用于获取命令行参数，同时可以设置默认值

可以获得各种类型的参数，无法获得无参数标记的参数

```Go

func main() {
// 一定要先执行 parse
flag.Parse()

// 获取参数
var port int
flag.IntVar(&port, "port", 8080, "server port")
}

```

## fmt

- `%+v` 格式符表示递归打印数据结构的字段名和值
- `%#v` 递归打印字段名和字段类型和值
- `%[2]v%[2]v%[1]v` 可以通过这种形式选定后边的第几个参数，避免重复传入相同参数，下标从 1 开始

  - 这种 template 还有个好处：如果后面的参数多于要求的，生成的字符串不会报错；而用"%v"的模板，多余的参数会已报错字符串的形式加入结果

- `%02d` 打印整形时前面补`0`，保留`2`位

### Scan

用于从字符串读取数据到变量，支持的类型包括基本类型，`[]byte`，`string`，`struct`，`slice`，`map`等

```Go
// 从 stdin 读入多个数据
a, b := 0, 0
fmt.Scan(&a, &b)

// 从 string 读入多个数据
var s string
var i int
Sscanf(" 1234567 ", "%5s%d", &s, &i)
```

### 重写 String 函数

- 可以编写自定义 String 函数，用于 fmt 打印

```Go
type MyStruct struct {
    Field1 string
    Field2 int
}

func (s MyStruct) String() string {
    // 在这里编写自定义的字符串表示逻辑
    return fmt.Sprintf("Field1: %s, Field2: %d", s.Field1, s.Field2)
}
```

## http

http 请求相关

```Go
req, err := http.NewRequest(http.MethodPost, "url", strings.NewReader("body"))
if err != nil {
  return err
}
req.URL.RawQuery = url.Values{"key":"value"}.Encode()
req.Header.Set("Content-Type", "charset=UTF-8")
req.SetBasicAuth("username", "password")

resp, err := http.DefaultClient.Do(req)
if err != nil {
  return err
}
defer resp.Body.Close()

body, err := io.ReadAll(resp.Body)
```

### 常见问题

- response 返回了 `\x1f\x8b\x08...` 乱码
  - 原因：服务端返回的响应头 Content-Encoding: gzip

## io

`func MultiWriter(writers ...Writer) Writer`
将多个 Writer 融合，每次写入会同时写入这些 Writer

`io.NopCloser(r io.Reader) io.ReadCloser`
自动判别输入的 reader 是否有对应的 writer 接口，如果没有则封装执行`Close()`时什么也不做

## io/ioutil

提供简易的 IO 操作

`ioutil.WriteFile`
简易写入文件

`ioutil.ReadFile`
简易读取文件

`ioutil.ReadDir`
读出文件夹中的子文件列表，列表按照名称排序，其中有可能包含文件夹元素

`ioutil.Discard`
用于抛弃读出的数据

```Go
// 有时收到 http response 后不关心 body 内容，可以直接丢弃
defer res.Body.Close()
_, _ = io.Copy(ioutil.Discard, res.Body)
```

## bufio

可以对 IO 进行方便的遍历

```Go
    // 初始化
		scanner := bufio.NewScanner(file)
		for scanner.Scan() {
      // 遍历每一行文本
			line := scanner.Text()
			log.Println(line)
		}

    // 遍历结束后，判断是否由于 error 导致了结束
		if err := scanner.Err(); err != nil {
			return err
		}
```

## log

log 相关

`log.Panic()`
输出 panic 级别日志后，开启 panic

`log.Fatal()`
输出 fatal 级别日志后，执行 os.Exit()

## os

`os.MkdirAll(path, os.ModePerm)`
创建一系列文件夹

`os.Getenv("HOME")`
获取环境变量，返回字符串

`func Getwd() (dir string, err error)`
获取当前当前目录的绝对路径

### os.signal

可以设置监听哪些系统信号

```Go
signals := make(chan os.Signal, 1)
signal.Notify(signals, os.Kill, syscall.SIGQUIT, syscall.SIGTERM, syscall.SIGINT)
select {
	case sig := <-signals:
		fmt.Println("get sys signal", sig)
		// end code
}
```

## path

提供路径相关操作

`path.Base(string)`
取得文件全名·

`path.Join([]string...)`
将 path 路径用'/'连接起来

- 注意，本函数只适合在 linux path 模式下，windows 下也只能增加'/'，某些情况会发生不兼容；另外对 url 使用时，也只适合 url host 后面的 path 部分

## net/url

`func Parse(rawurl string) (*URL, error)`
Parse 函数可以从字符串解析出 url.URL 的对象指针，url.URL 包含了 URL 的各参数

## pool

提供对象池接口，用于缓存对象，避免频繁申请、释放内存造成的性能问题。

- 一定要创建 Pool 对象时执行`New`如果池中没有对象，会调用指定的 New 方法生成一个；如果没有指定 New 方法，那么返回 nil
- 池不可以指定大小，大小受限于 GC 的临界值
- 对象的最大缓存周期是 GC 周期，当 GC 调用时没有引用的对象会被清理掉
  - pool 包在 init 中注册一个 poolCleanup 函数用于清除所 pool 里的缓存对象，每次 GC 之前会被调用
- Get 方法返回时是返回池中任意一个对象，没有顺序；

## math

`func Sqrt(x float64) float64`

### code snippet

```Go
func RoundFloat(val float64, precision uint) float64 {
	ratio := math.Pow(10, float64(precision))
	return math.Round(val*ratio) / ratio
}
```

## math/rand

用于生成随机数，注意和"crypto/rand"包名相同

- 使用前要先做一个种子，否则其实不随机。
  `rand.Seed(time.Now().Unix())`

`func Float32() float32`/`func Float64() float64`
return a pseudo-random number in [0.0,1.0)

`func Intn(n int) int`/`func Int31n(n int32) int32`
return a non-negative pseudo-random number in [0,n)

## reflect

反射相关

`func DeepEqual(x, y interface{}) bool`
利用反射对两个对象深度比较，比较 interface 类型时，必须类型和值都相同才能 true。

- 一些特殊的比较点：
  - Func 类型如果都为 nil 则 equal，否则 not equal。即使是相同函数都 not equal
  - "empty slice" != "nil slice"
  - slice 不比较 cap
  - 如果对象内存在环状依赖，算法会自动识别出这种依赖，在第二次遇到重复对象时会自动认为 equal

## regexp

正则表达式类

```Go
// 匹配找到分组，分组下标按照可能找到的位置排列，第 0 位置是整个匹配的字符串
re := regexp.MustCompile(`pattern`)
var reRet []string = re.FindStringSubmatch(`string`)
matches := regex.FindAllStringSubmatch(`string`, -1)
```

## runtime

运行时调用相关

`runtime.NumCPU()`
返回当前硬件的 CPU 核数

`runtime.GOMAXPROCS(int)`
设置当前可同时运行的最大线程数(GPM 中 P 的数量)

`var numCPU = runtime.GOMAXPROCS(0)`
返回用户指定的最大 P 数量。如果没有设置，默认值为 CPU 核数

`runtime.Caller(skip int) (pc uintptr, file string, line int, ok bool)`
返回调用栈帧信息，`skip >= 0`。如果是 0，表示打印调用`Caller`的这行代码信息。

### runtime/trace

to generate traces for the Go execution tracer
The execution trace captures a wide range of execution events such as goroutine creation/blocking/unblocking,
syscall enter/exit/block, GC-related events, changes of heap size, processor start/stop, etc.
A precise nanosecond-precision timestamp and a stack trace is captured for most events.
The generated trace can be interpreted using `go tool trace`.

- `func Start(w io.Writer) error`
  开始将 trace 信息写入文件

## sort

提供排序相关接口

```Go
func Sort(data Interface)	// 快排
func Stable(data Interface)	// 稳定排序
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
```

- `SearchInts(a []int, x int) int`
  在有序数组中，用二分查找法，找到目标第一次出现的下标。如果没有找到，则返回将插入的位置下标，即下一个元素第一次出现的下标。

### sort.Ints

`func Ints(x []int)`
对[]int 进行排序

### sort.Reverse

`func Reverse(data Interface) Interface`
适配 `sort.Interface.Less` 为反向处理(`<` 变 `>`)

`type IntSlice []int`
为`[]int`附加`sort.Interface`，以便利用各种 Sort 中的函数

```Go
s := []int{5, 2, 6, 3, 1, 4} // unsorted
sort.Sort(sort.Reverse(sort.IntSlice(s)))
fmt.Println(s)	// output: 6 5 4 3 2 1
```

### sort.Search

`func Search(n int, f func(int) bool) int`
LowerBound 算法，在有序数组中查找`[0,n)`范围内满足`f`的最小下标，如果没有满足条件的，则返回 n

- 利用闭包实现`f`
- 如果二分查找范围是`[x, x+n)`，则可以`Search(n,...)`，然后在`f`函数进入后`index+x`修正范围

### sort.Slice

`func Slice(x interface{}, less func(i, j int) bool)`
利用闭包对任意类型的 Slice 排序。该函数底层利用反射实现

### sort.Reverse

将接口的 less 函数结果取反，以便反转最终排序结果达到反向排序(从大到小)的目的

`func Reverse(data Interface) Interface`

```Go
s := []int{5, 2, 6, 3, 1, 4} // unsorted
sort.Sort(sort.Reverse(sort.IntSlice(s)))
```

## sync.Mutex

互斥锁

- Go 中的 Mutex 是不可重入的，不像 Java 中的
- Mutex 是一个 Struct 对象，并且其中的私有成员变量有重要作用，所以 Mutex 对象使用中不能复制拷贝；sync.WaitGroup 也是相同的原理。

- 为了避免对 Mutex 拷贝，`go vet`会检查 sync.Mutex 对象的拷贝并报错，其原理是利用`noCopy`内嵌对象来进行检测。
  其他 struct 如果不想被拷贝，可以利用内嵌`type DoNotCopy [0]sync.Mutex`(GRpc 实现)，
  由于数组 len 为 0 不占空间(不要放在最后一个元素，否则占 8 字节)

### 实现设计点

- 基本的原子操作和判断是利用`atomic.CompareAndSwapInt32`
- 锁定后唤醒，是利用`sync.runtime_SemacquireMutex`
- 锁有几种状态：未加锁、已锁定、饥饿状态
- "饥饿状态"激发条件：队首元素等待时间超过 1ms。激发后，新的 lock 请求会被自动排到队尾，不进行自旋等待。
- "饥饿状态"使得 Lock 排队中的 goroutine 得以抢到锁，以避免"尾部延迟现象"
- 新抢锁的 goroutine，如果满足下面四个条件，会尝试自旋优先抢锁
  1. mutex 已经被 locked 了，处于正常模式下；
  2. 当前 goroutine 为了获取该锁进入自旋的次数小于四次；
  3. 当前机器 CPU 核数大于 1；
  4. 当前机器上至少存在一个正在运行的处理器 P 并且处理的运行队列为空；

## sync.WaitGroup

可以为多个 goroutine 设置统一起跑时机或统计统一结束时机。

## sync.Map

实现了线程安全的 Map

## strings

字符串操作函数

`func ToLower(s string) string`

- `strings.EqualFold(a, b string) bool`
  忽略大小写下，是否相等

`strings.Split(str, seperator string) []string`
将字符串按照分隔符切割为多个子字符串组成的切片

`strings.Builder`
用于字符串拼接`builder.WriteString("test")`，在大量字符串拼接时性能远高于`+`，只在最后`builder.String()`时申请一次内存。

`func NewReader(s string) *Reader`
将 string 转为 io.Reader

## strconv

字符串和各种类型间的转换

- `func FormatFloat(f float64, fmt byte, prec, bitSize int) string`
  把 float 类型转为字符串。有时涉及浮点位数的处理，如"获取浮点数小数点后位数"，可以先转为字符串再处理，否则容易受浮点精度问题影响。
  - fmt 输出格式，如果想输出所有小数不想被 e 替换，可以用`f`格式
  - prec 精度，默认 `-1`

## time

提供时间日期相关操作

- 如果想在某个时间基础上减掉一段时间，可以用 Add 负数的方式。

```Go
str := "2019-12-28 10:01:02.123456789"
timeLayout := "2006-01-02 15:04:05.000000000"
t := time.Parse(timeLayout, str)
```

将字符串按照模版字符串的格式解析为 time.Time 对象

- 其中模板字符串比较特殊，必须是指定的常量或`"2006-01-02 15:04:05.000000000"`这个值的变式
- 时间的秒单位后，用`0`或`9`表示一个位置，精度可以到纳秒(9 个 0)。

- 由于对象内部构造细节，无法直接比较，time 的比较要用专用函数`after/before`，相等也要用`func (t Time) Equal(u Time) bool`

- `func (t Time) Equal(u Time) bool`
  比较两个 Time 对象是否相等，会考虑到时区问题。注意用`t1 == t2`的方式比较可能存在问题。

### ticker

- 使用 ticker 时，可能由于 ticker 设置时间较长同时没有及时关闭而导致的 ticker 泄漏，可以用`defer ticker.Stop()`解决。
- 注意 Stop ticker 后，ticker.C 并不会关闭，只是不再有值返回了。

```Go
// ticker 结合 for select 的用法
func testFunc(){
  ticker := time.NewTicker(time.Second)
  defer ticker.Stop()
  doneCh := make(chan struct{})
  defer close(doneCh)
  go func() {
  waitLoop:
    for {
      select {
      case <-doneCh:
        // 注意，这里直接用 break 的话，由于被 select 吸收，所以永远无法退出循环
        break waitLoop
      case <-ticker.C:
        // do something
        // 注意，如果有多个 case，并且想保持循环，要用 continue 继续循环，不小心 return error 就会跳出循环
      }
    }
  }()
  // some code
}
```

- 对于等待时间较短的，可以利用`time.After`实现简易等待，但是如果频率高、时间长，可能会造成 ticker 泄漏

```Go
select {
  case <-time.After(time.Second):
    // do something
  case ...:
}
```

## unicode

`func IsDigit(r rune) bool`
`func IsLetter(r rune) bool`

## unsafe

底层指针相关的操作，可以调用 C 语言库函数，就是所谓的 CGo 机制

- CGo 只能调用 C 的接口，无法调用 C++。由于 C++标准太多太复杂，无法支持。可以通过 C 封装 C++的方法实现间接调用。

`unsafe.Sizeof(x)`
传入任意变量，返回该变量对应类型所占字节数

# 第三方

## "go.uber.org/atomic"

把原有的 atomic 函数封装成方便使用的类，如`atomic.Int64`

## "github.com/sourcegraph/conc"

便于并发的处理

```go
import (
  "github.com/sourcegraph/conc"
  "github.com/sourcegraph/conc/pool"
  "github.com/sourcegraph/conc/panics"
)


func main() {
  for i := 0; i < 10; i++ {
		wg.Go(func() {
      //...
		})
	}
  // 所有 goroutine 结束才返回
  // 如果某一 groutine 发生 panic 则会被捕获到
	pan := wg.WaitAndRecover()

  ctx := context.Background()
	p := pool.NewWithResults[string](). // 创建可返回结果的携程池
		WithMaxGoroutines(2). // when no more worker, new will be blocked
		WithContext(ctx)      // when first error happend, all will be cancel
  for i := 0; i < 10; i++ {
    p.Go(func(c context.Context) (ret string, err error) {  // 执行
      // ...
    })
  }

	// if panic happend in one of the goroutine, p.Wait() will cause panic
	pc.Try(func() { rets, err = p.Wait() })
	// pc.Repanic()	// re propagate panic
	p1 := pc.Recovered() // catch panic
	fmt.Println(rets, err, p1)
}
```

## github.com/jinzhu/copier

利用反射嵌套拷贝结构体间的同名成员

- 大类型必须一致(比如 int32 和 int64 大类型一致)，否则不能自动做类型变更

## github.com/go-co-op/gocron

定时任务

```go

func main() {
  	scheduler := gocron.NewScheduler(time.UTC)
    s.CronWithSeconds("*/1 * * * * *").Do(task) // every second
}
```

## github.com/jmoiron/jsonq

方便的对 json/yaml 深度嵌套结构进行查询，通常先将 json/yaml 解析为`map[string]interface{}`类型，然后再用 jsonq 查询

```Go
import (
	"strings"
	"encoding/json"
	"github.com/jmoiron/jsonq"
)

data := map[string]interface{}{}
dec := json.NewDecoder(strings.NewReader(jsonstring))
dec.Decode(&data)
jq := jsonq.NewQuery(data)

// data["subobj"]["subarray"][1] -> 2
jq.Int("subobj", "subarray", "1")
```

## github.com/wI2L/jsondiff

比较两个 json 字符串，返回差异。可以忽略多余的空格等。

```Go
patch, err := jsondiff.CompareJSON(str1, str2)
if err != nil {
}
log.Pringln(patch)
```

## github.com/go-redsync/redsync/v4

redis 分布式锁

## github.com/avast/retry-go

方便的进行 retry

```Go
import "github.com/avast/retry-go"

func target() (string, error) {
  return "", fmt.Errorf("error")
}

func main() {
  var ret string
  var err error
  retry.Do(func() error {
    ret, err = target()
  }, retry.Attempts(3))

  if err != nil {}
  fmt.Println(ret)
}
```

## github.com/google/uuid

提供 UUID 生成功能

`log.Println(uuid.New().String())`

## github.com/davecgh/go-spew/spew

递归打印复杂嵌套对象

`spew.Printf("myVar1: %v -- myVar2: %+v", myVar1, myVar2)`

## https://github.com/Velocidex/ordereddict

模仿 Python 的 collections.OrderedDict 类，以插入顺序为输出顺序，可以 map 避免乱序的问题。

## https://github.com/kisielk/godepgraph

可以打印 Go 的包依赖关系，依赖 Graphviz 组件可以进一步生成图。

`godepgraph ./pkg/api`

## golang.org/x/text

实现了各种文本编码、字符集相关操作

## github.com/jmoiron/sqlx

实现了各种方便的 SQL 查询

```Go
// 对指定的 id 数组进行查询
ids := []int{1,2,3}
query, args, err := sqlx.In("select * from xxx where id in (?);", ids)
if err != nil {
  return err
}
query = bm.tx.Rebind(query)

if err := bm.tx.SelectContext(ctx, &res, query, args...); err != nil {
  return err
}
```

## gopkg.in/yaml.v3

yaml 操作，用法和 json 包类似。v3 版本修复`map[interface{}]interface{}`的类型问题。

```Go
if err := yaml.Unmarshal([]byte(str), &config); err != nil {
}
```

## snippets

```Go
func MysqlRealEscapeString(value string) string {
    var sb strings.Builder
    for i := 0; i < len(value); i++ {
        c := value[i]
        switch c {
        case '\\', 0, '\n', '\r', '\'', '"':
            sb.WriteByte('\\')
            sb.WriteByte(c)
        case '\032':
            sb.WriteByte('\\')
            sb.WriteByte('Z')
        default:
            sb.WriteByte(c)
        }
    }
    return sb.String()
}
```
