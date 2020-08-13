---
layout: post
title: "Go Package Notes"
date: 2019-12-28 22:00:00 +0800
tags: Go
---

Go 常用包的功能，及注意点

## flag

用于获取命令行参数，同时可以设置默认值

可以获得各种类型的参数，无法获得无参数标记的参数

## fmt

`%+v` 格式符表示将数据结构打印出来
`%[2]v%[2]v%[1]v` 可以通过这种形式选定后边的第几个参数，避免重复传入相同参数

## io/ioutil

提供简易的 IO 操作

`ioutil.WriteFile`
简易写入文件

`ioutil.ReadFile`
简易读取文件

`ioutil.ReadDir`
读出文件夹中的子文件列表，列表按照名称排序，其中有可能包含文件夹元素

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

## os.signal

可以设置监听哪些系统信号

## path

提供路径相关操作

`path.Base(string)`
取得文件全名·

## pool

提供对象池接口，用于缓存对象，避免频繁申请、释放内存造成的性能问题。

- 池不可以指定大小，大小受限于 GC 的临界值
- 对象的最大缓存周期是 GC 周期，当 GC 调用时没有引用的对象会被清理掉
  - pool 包在 init 中注册一个 poolCleanup 函数用于清除所 pool 里的缓存对象，每次 GC 之前会被调用
- Get 方法返回时是返回池中任意一个对象，没有顺序；如果池中没有对象，会调用指定的 New 方法生成一个；如果没有指定 New 方法，那么返回 nil

## rand

使用前要先做一个种子，否则其实不随机。
rand.Seed(time.Now().Unix())

## regexp

正则表达式类

```Go
var reRet []string = regexp.MustCompile(`pattern`).FindStringSubmatch(`string`)
```

## sync.WaitGroup

可以为多个 goroutine 设置统一起跑时机或统计统一结束时机。

## strings

字符串操作函数

`strings.Split(str, seperator string) []string`
将字符串按照分隔符切割为多个子字符串组成的切片

## time

提供时间日期相关操作

- 如果想在某个时间基础上减掉一段时间，可以用 Add 负数的方式。

`t := time.Parse("2006-01-02 15:04:05", str)`
将字符串按照模版字符串的格式解析为 time.Time 对象

- 其中模板字符串比较特殊，必须是指定的常量或`"2006-01-02 15:04:05"`这个值的变式。
- 这里精度只能到秒，毫秒、微秒、纳秒可以用简单的字符串转换完成解析。

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

## unsafe

底层指针相关的操作

`unsafe.Sizeof(x)`
传入任意变量，返回该变量对应类型所占字节数

## golang.org/x/text

实现了各种文本编码、字符集相关操作
