---
layout: post
title: "VSCode Go remote debug"
date: 2024-01-07 22:00:00 +0800
tags: Go IDE VSCode
---

![debug](/assets/images/2024-01-07-VSCode_remote_debug_1.png)

记录用 VSCode 和 Docker 实现 Go 跨操作系统(Mac, Linux)远程调试的方法。

# 原理

- 基本的 Debug 原理参考[应用程序调试原理浅析](https://mp.weixin.qq.com/s/M3ZroS_01ej0fm56XHiqQA)

  - Debug 的关键是"断点陷入"和"寄存器查看"，需要 CPU 指令集和操作提供同时支持

- VSCode 的 Go extension(的 delve 调试器) 基于[DAP 协议](https://microsoft.github.io/debug-adapter-protocol/overview)实现调试功能

  - DAP 是一个 UI 调试协议，描述了断点、源码查看、多线程、复杂结构体查看、表达式查看等实现细节

- debugAdapter 有`legacy`模式和`dlv-dap`模式

  - 尽量使用新版的`dlv-dap`模式，对各种功能支持较完善

- 从 VSCode 发起调试有两种模式
  - `attach`适合已经在 linux 启动目标进程、附加调试的情况
  - `launch`从 VSCode 发起新进程的启动，调试结束后自动退出。这种模式可以在远程主机上启动任何进程，使用时要小心安全问题。

# 步骤

- 前提：

  - docker 镜像是已经安装好 Go 环境的，例如`golang:1.18-alpine`
  - 本机 VSCode 已装好，并可以 debug 本地 Go 工程，Go 工程文件夹名为`GoHello`

- 启动 docker 镜像`docker run -it -p 12345:12345 -v /usr/src/GoHello:/xxx/GoHello --name remote_debug_test golang:1.18-alpine bash`

  - 设定 debug 用端口`12345`
  - 设定文件夹映射

- 在 container 中安装 dlv

  - 最简单的安装方法是利用 Go 编译，然后拷贝到目标 path 下

  ```s
    $ GO111MODULE=on GOBIN=/tmp/ go install github.com/go-delve/delve/cmd/dlv@master
    $ mv /tmp/dlv $GOPATH/bin/dlv-dap
  ```

- 在本机 Go 工程中配置`.vscode/launch.json`文件，这里以`launch`模式为例：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "remote debug launch",
      "type": "go",
      "debugAdapter": "dlv-dap",
      "request": "launch",
      "mode": "exec",
      "port": 12345,
      "host": "127.0.0.1",
      "program": "/usr/src/GoHello/hello",
      "logOutput": "dap",
      "showLog": true,
      "stopOnEntry": false,
      "apiVersion": 2
    }
  ]
}
```

- 编译代码为 linux 可执行二进制文件`GOOS=linux go build -gcflags=all="-N -l" -o hello hello.go`

- 在 terminal 中启动 dlv-dap `dlv-dap dap --listen=:12345 --log`

- 在 VSCode 代码界面添加断点，然后在 Debug 中选择"remote debug launch"启动，就可以正常断点了

# 补充

- `attach`模式的`.vscode/launch.json`配置不同点：

  - `request`为`attach`
  - `mode`为`remote`
  - `stopOnEntry`为`false`

- 设置`"stopOnEntry": true`时，可能看到调试页面报错"unknown goroutine 1"。
  - 这是符合预期的，启动时确实没有可显示的调用栈，但是 VSCode 要求至少一个，所以报错了，不影响后面正常调试

# 遗留问题

- [ ] 日志只能在 terminal 中看到，无法在 VSCode 的"DEBUG CONSOLE"中看到
- [ ] 为什么编译要加`-gcflags=all="-N -l"`
- [ ] docker 在 mac 默认不支持 ipv6

# 参考

- [VSCode 调试界面](https://code.visualstudio.com/docs/editor/debugging)
- [launch.json 可配置项](https://github.com/golang/vscode-go/wiki/debugging#launchjson-attributes)
