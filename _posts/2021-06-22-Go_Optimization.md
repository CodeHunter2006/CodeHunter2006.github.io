---
layout: post
title: "Go 优化"
date: 2021-06-22 23:00:00 +0800
tags: Go
---

# Flame Graph(火焰图)

当程序出现性能问题，这时想找到哪个函数比较耗时，火焰图就可以分析函数对 CPU 的占用情况。

在生成火焰图之前，先要用 Linux 自带的 CPU 性能分析工具 perf 抓取数据，
它会每秒执行 99 次，每次会抓取占用当前 CPU 的函数名以及调用栈，生成在文件中。
由于生成的文件过大并不直观，所以我们需要用工具分析文件进一步生成一个 SVG 格式的火焰图。

![Flame Graph](/assets/images/2021-06-10-Go_Optimization_1.jpeg)

- 火焰图的读法：

  - Y 轴表示函数调用栈，越被晚调到的函数在最上层
  - X 轴表示调用时间，如果多次采样都是同一个函数，则时间累加。注意从左向右不是时间顺序关系，而是合并栈后的函数名顺序排列
  - 我们在途中尽量寻找一个**较宽**的区域(plateaus 平顶)，该区域就是占用 CPU 较长的函数
  - 图的颜色是没有意义的，由于名为"火焰图"(下宽上窄像火焰)，所以通常是暖色调
  - SVG 是带信息的矢量图，所以可以有一定的互动：缩放、鼠标悬浮显示函数名、搜索函数名

- Go 生成火焰图/方块图

  - `runtime/pprof` 可以生成 cup/mem 的采样数据
  - `net/http/pprof` 对 `runtime/pprof` 进行了包装，对外提供 http 数据采集接口

- 下面代码可以直接生成 cpu/mem 快照到文件

```Go
func main() {
    var cpuProfile = flag.String("cpuprofile", "", "write cpu profile to file")
    var memProfile = flag.String("memprofile", "", "write mem profile to file")
    flag.Parse()
    //采样cpu运行状态
    if *cpuProfile != "" {
        f, err := os.Create(*cpuProfile)
        if err != nil {
            log.Fatal(err)
        }
        pprof.StartCPUProfile(f)
        defer pprof.StopCPUProfile()
    }

    var wg sync.WaitGroup
    wg.Add(100)
    for i := 0; i < 100; i++ {
        go workOnce(&wg)
    }

    wg.Wait()
    //采样memory状态
    if *memProfile != "" {
        f, err := os.Create(*memProfile)
        if err != nil {
            log.Fatal(err)
        }
        pprof.WriteHeapProfile(f)
        f.Close()
    }
}
```

- 然后执行`go tool pprof cpu.pprof`就可以分析结果。
- 上面的文本结果不易观察，在 Mac 上可以先安装`brew install graphviz`，然后在`pprof`命令输入子命令`web`可以打开 SVG 图：

![方块图](/assets/images/2021-06-22-Go_Optimization_2.jpg)

- 如果是 cpu 方块图，图中方块的大小表示 cpu 占用时间越长
- 如果是 mem 方块图，图中方块大小表示 mem 占用容量

- 用下面语句可以开启 http 接口实现动态采样

```Go
package main

import (
    "time"
    "net/http"
    _ "net/http/pprof"  // 注意这里一定要初始化包
)

func main() {
    go SomeFunction()
    http.HandleFunc("/get", httpGet)

    http.ListenAndServe("localhost:8000", nil)  // 开启 pprof 监听
}
```

- 可以执行命令`go tool pprof main http://localhost:8000/debug/pprof/heap`采集，
  然后生成 SVG 分析
- 用 Uber 开源的火焰图工具`go-torch`可以将数据生成火焰图，其实方块图和火焰图类似，都能快速直观定位问题
