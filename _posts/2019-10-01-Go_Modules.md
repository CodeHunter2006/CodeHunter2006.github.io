---
layout: post
title: "GoModules基本使用方法"
date: 2019-10-01 10:00:00 +0800
tags: Go
---

![Go Modules](/assets/images/2019-10-01-Go_Modules_1.png)
Go Modules 是 Golang 1.11 版本时加入的模块依赖管理解决方案，解决对 VCS 的依赖。在 1.13 版本发布时加入了 Go Proxy，原来的目的是通过代理加速拉取和 build 速度，但是间接的解决了国内的拉取问题，可以 **很方便** 的下载依赖包，目前已经比较成熟，值得引入项目使用。

# 涉及的概念

Modules(模块)是可以被依赖的代码单元，可以包含多个 Package(包)。

`go.mod`文件用于描述当前项目依赖哪些模块及依赖方式。每个被依赖的模块会记录版本号、地址、备选版本、不选版本等信息，以便未来通过网络下载构建时能够精确产生出最终编译结果。

Go Proxy 用于依赖包的查找和下载，可以备选多个代理网址。不同项目依赖的相同模块在本地只存储一份，避免重复存储导致容量暴涨问题(比如 node.js 的 npm)。

`go.sum`(checksum database)文件用于存储各个 module 精确版本号(类似 Python 的 requirements.txt 或 Ruby 的 Gemfile.lock)及校验信息，以避免依赖的模块被篡改。

`go mod`命令用于模块管理，可以独立运行，也集成在 go get、go build、go test 等各种命令中。

# 环境变量

## GO111MODULE=auto

开关环境变量，表示是否采用 Go modules，可选项`auto/on/off`

- auto 项目中包含`go.mod`文件时才启用 Go modules
- on 启用 Go modules
- off 不启用，仍然使用 GOPATH
- 默认值`auto`

## GOPROXY=https://proxy.golang.org,direct

用`,`分割的 Go module proxy 列表

- 如果上一个代理连接时遇到 404 或 410，则自动尝试下一个，遇到 direct 后会终止并回源(回到源地址下载)
- 如果值填写`off`则不允许 Go 从任何地方拉取版本

国内可选的代理：

- 七牛云 https://goproxy.cn
- 阿里云 https://mirrors.aliyun.com/goproxy/
- https://goproxy.io/

## GONOPROXY GONOSUMDB GOPRIVATE

用于指定哪些模块不要走 Go Proxy，通常只用 GOPRIVATE 就可以。

# 配置文件

## go.mod 语法

- module 定义当前项目的模块路径
- go 设置当前编译采用的 Golang 版本
- require 依赖一个特定的模块版本
- exclude 排除特定的模块版本
- replace 将一个模块版本替换为另一个特定的模块版本(例如测试用 stub)

## go.sum 语法

```
xxx.com/xxx v0.1.1 h1:xxxxxxx
```

域名/模块路径(或 go.mod 文件路径) [空格] 版本号 [空格] 对模块 SHA-256 生成的 Hash 值

# 文件存储

在使用模块的时候，GOPATH 是无意义的，不过它还是会把下载的依赖储存在 GOPATH/src/mod 中，也会把 go install 的结果放在 GOPATH/bin（如果 GOBIN 不存在的话）

# 命令

- `go mod download`
  下载模块到本地缓存，缓存路径是`$GOPATH/pkg/mod/cache`
- `go mod edit`
  是提供了命令版编辑 go.mod 的功能，例如`go mod edit -fmt go.mod` 会格式化 go.mod
- `go mod graph`
  把模块之间的依赖图显示出来
- `go mod init`
  初始化模块（例如把原本 dep 管理的依赖关系转换过来）
- `go mod tidy`
  增加缺失的包，移除没用的包
- `go mod vendor`
  把依赖拷贝到`vendor/`目录下
- `go mod verify`
  确认依赖关系
- `go mod why`
  解释为什么需要包和模块

# 其他问题

## 如何用 Go Modules 替代 GOPATH ?

- 使用 Go 1.13，对 Go Modules 支持较好
- 使用 GOPATH 环境变量的升级替代方案：
  - 设置 GOBIN，避免依赖 GOPATH/bin
  - 设置 GO111MODULE 为 on
  - 设置 GOPROXY 为国内可用代理
- 项目不需要在放在 GOPATH/src 下
- 在项目根目录执行`go mod init 模块路径`，以生成 go.mod 文件

## 如何拉取私有模块 ?

TODO 待补充

## 常见问题

- 如果遇到编译问题，可以执行`go clean -modcache`清空缓存的模块，重新下载。
- `go get xxx.com/xxx/xxx@version`，get 命令增加了版本选项，默认会查找名为`latest`的 tag，然后查找 master 的最新 commit。也可以执行版本号(形为`vx.x.x`的 tag)或者 Hash。
- `go get -u all`更新所有模块及其单元测试，不会更新主版本号。
- 使用了 Go Modules 之后，import 包时的路径对大小写敏感，如果之前大小写不规范可能报找不到包的错误。
- Go Modules 目前和 GitLab 有些兼容问题，可能引发问题。
- 使用 Go Proxy 之前，如果没有梯子，可以安装`github.com/gpmgo/gopm`，用`gopm get -g xxx`代替`go get xxx`
- 另外，如果没有梯子，`https://github.com/golang`是`https://golang.org`的镜像，可以手动下载
- Go Modules 下编译动态库(.so)会有问题，目前 Go 的项目是以源码依赖编译为主的，在硬盘、内存没有因为可执行文件体积过大而产生问题前，不需要考虑动态库。
