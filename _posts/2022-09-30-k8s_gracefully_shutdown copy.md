---
layout: post
title: "K8S Gracefully Shutdown"
date: 2022-09-29 22:00:00 +0800
tags: Go K8S
---

![Sleep](/assets/images/2022-09-30-k8s_gracefully_shutdown_1.webp)

实际开发中，我们经常会遇到需要程序优雅退出的情况，被打断的程序可能造成严重的后果。
这里记录 K8S 及程序如何配合实现优雅退出。这里以 http 短连接为例。

## 基本结构

- 部署结构：deploy -> replicaSet -> pod -> container -> process

  - deploy 的 `strategy.type` 要用 `RollingUpdates`，这样可以在多实例下保持始终有 live 的 pod

- 调用链路：client -> ingress -> svc -> pod -> process -> handler
  - 进行退出时，要先从上游 svc 中把当前 pod 摘掉，避免新的请求打到当前 pod，然后逐步结束处理后退出

## 基本流程

1. 发起新的 deploy，开启`RollingUpdates`过程，选择一部分 pod 进行关闭
2. pod 向其中的 container 下的主进程发送`SIGTERM`信号
3. 进程收到`SIGTERM`后开始进行退出处理
4. 进程正常退出、container 退出、pod 退出
5. 如果进程未能正常退出，超过了等待时间，则 pod 下发`SIGKILL`信号，强制杀掉进程
6. pod 完成关闭，启动新的 pod，继续`RollingUpdates`

- K8S 默认会发送`SIGTERM`信号，如果需要修改信号类型，可以利用`container.lifecycle.preStop.exec.command`指定命令发送
- 也可以利用`preStop`脚本进行其它类似关闭的动作

## K8S 配置

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 10
  strategy:
    type: RollingUpdates
    rollingUpdate:
      maxSurge: 2 # 最大额外可以存在的副本数，可以为百分比，也可以为整数
      maxUnavaliable: 2 # 最大不可用状态的pod的最大值，可以为百分比，也可以为整数
  containers:
    - name: test
      lifecycle:
        preStop:
          exec:
            command: ["kill", "-1", "1"] # 指定启动关闭流程时 container 执行的命令
  terminationGracePeriodSeconds: 120 # 优雅退出发送
```

- `command` 是字符串数组，用于拼接执行的命令
- 一般 container 内的 PID 是 1，所以用上面命令即可发送`SIGHUP(-1)`到进程

## Go 代码示例

```Go
package main

import (
        "fmt"
        "os"
        "os/signal"
)

func main() {
	log.Println("start")
	defer log.Println("end")

	sig := make(chan os.Signal)
	signal.Notify(sig)

outer:
	for s := range sig {
		select s {
		case syscall.SIGTERM:
			log.Println("directly end with", s)
			return
		case syscall.SIGHUP:	// gracefully shutdown
			log.Println("start gracefully shutdown with", s)
		    break outer
		default:
		  log.Println("unknow signal", s)
		}
	}

	gracefulShutdown()
}

var gTaskWG *sync.WaitGroup = new(sync.WaitGroup)
func gracefulShutdown() {
	const totalWaitTime = time.Second * 30	// 这里的时间一定要比 K8S 设定的 terminationGracePeriodSeconds 短
	const logTickInterval = time.Second

	finshCh := make(chan struct{})
	logTicker := time.NewTicker(logTickInterval)
	defer logTicker.Stop()
	timeOutTicker := time.NewTicker(totalWaitTime)
	defer timeOutTicker.Stop()

	go func() {
		gTaskWG.Wait()
		close(finshCh)
	}

	for {
		select {
		case <-finshCh:
			log.Println("finished all tasks")
			return
		case <-timeOutTicker:
		    log.Println("timeout")
			return
		case <-logTicker:
		    log.Println("still waiting")
		}
	}
}

func addOneTask() (doneFunc func()) {
	gTaskWG.Add(1)
	return func(){
		gTaskWG.Done()
	}
}

func businessFunc() {
	doneFunc := addOneTask()
	go func(){
		defer doneFunc()
		// ... do something
	}
}
```

## 常见问题

- signal 被"吸收"的问题
  - 有时启动进程不是通过命令，而是通过脚本执行的，这时如果向脚本进程发送 signal 会被吸收掉
    - 这时启动命令应该尽量将进程本身暴露在最外层，这样可以直接捕获到 signal
  - 有时启动进程会以同步方式启动子进程，这种情况下子进程会捕获 signal
    - 所以要在子进程里增加处理，并在进程结束时返回特定的 exitCode，以便父进程继续处理
