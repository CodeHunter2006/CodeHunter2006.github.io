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

- 文件夹、package、文件名之间是没有固定关系的
- 一个文件夹下只能有一个 package
- 如果`package main`，表示其中的`main`函数是程序入口函数，那么这个文件夹可以编译为一个可执行文件(可能依赖外部文件)
- 一个 Module 的地位和文件夹相同，只不过在这个文件夹根目录放着`go.mod`和`go.sum`，并且这个文件夹还会作为 github 的项目根文件夹
- `import "xxx"`导入的本质是导入一个**文件夹**，如果这个文件夹在本地没有而需要从 github 导入，则自动`go get`对应的 module

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
  - 每次 Go 在新版中加入新 API 后，如果在代码中使用了，就得在这里声明最低的版本号
- require 依赖一个特定的模块版本
- exclude 排除特定的模块版本
- replace 将一个模块版本替换为另一个特定的模块版本(例如测试用 stub)，替换的目标类型：
  - 这个 module 的另一个版本号
  - 另一个 module
  - 也可以是一个路径指向的本地 module

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

利用 replace 设置好本地工程路径在当前项目中的依赖关系，然后执行`go mod vendor`

## 如果有依赖版本冲突怎么办？

比如当前项目依赖 A、B 两个模块，A 依赖 C(V1)、B 依赖 C(V2)，如果保证编译时能够引用到正确的 C？

- 方案：
  如果 C 的 V1 和 V2 是两个主版本，则不会产生分歧，会被认为是不同库；
  如果 V1 和 V2 是两个子版本，则需要 A B 各自的 go.mod 文件中指定精确版本号，这样就可以正确编译了。
  如果 V1 和 V2 是两个子版本，但是没有精确指定版本号，则会自动拿其中最新的主版本编译。

## 是否采用 vendor 锁定版本？

一般局域网 gitLab 或代理可靠的情况下，在 go.mod 中指定明确的版本号即可锁定，不需要维护 vendor。

## module 的版本号语义

我们用三级版本号表示一个 module 的版本：
`vMAJOR.MINOR.PATCH`

- v：表示版本
- MAJOR：主版本，通常是大版本升级，导致向前不兼容
- MINOR：次版本，通常是向下兼容的 feture
- PATCH：修订版本，如一些 bugfix

默认情况下执行`go get`是不会升级主版本号的，因为大版本不兼容。需要执行`go get -u`进行升级。
如果想升级下小版本，可以主动指定版本`go get xxx@vx.x.x`

在版本升级时，会在 github 中打 tag，比如`v3.0.0`。
import module 时，可以在后面跟大版本号，而代码中继续直接使用包名即可，如：

```Go
import "xxx/abc/v3" // v3.0.0

func main() {
  abc.Func1() // 继续使用包名
}
```

## 常见问题

- go mod init 在没有接 module 名字的时候是执行不了的，会报错 go: cannot determine module path for source directory。可以这样执行：`$ go mod init github.com/jiajunhuang/hello`
- 如果遇到编译问题，可以执行`go clean -modcache`清空缓存的模块，重新下载。
- `go get xxx.com/xxx/xxx@version`，get 命令增加了版本选项，默认会查找名为`latest`的 tag，然后查找 master 的最新 commit。也可以执行版本号(形为`vx.x.x`的 tag)或者 Hash。
- `go get -u all`更新所有模块及其单元测试，不会更新主版本号。
- 使用了 Go Modules 之后，import 包时的路径对大小写敏感，如果之前大小写不规范可能报找不到包的错误。
- 使用 Go Proxy 之前，如果没有梯子，可以安装`github.com/gpmgo/gopm`，用`gopm get -g xxx`代替`go get xxx`
- 另外，如果没有梯子，`https://github.com/golang`是`https://golang.org`的镜像，可以手动下载
- Go Modules 下编译动态库(.so)会有问题，目前 Go 的项目是以源码依赖编译为主的，在硬盘、内存没有因为可执行文件体积过大而产生问题前，不需要考虑动态库。
- 私有包如果不想发布到网上，需要手动添加 require ，然后 replace 进行替换，将私有包指向本地 module 所在的绝对或相对路径。一般用相对路径更通用。
- 使用 replace 时要非常小心，如果有 A B 两个模块分别依赖 C 的两个版本，而 replace 只重定向了其中一个版本，可能会导致版本问题而编译不过。
- 如果在本项目中需要同时使用同一个包的不同版本，可以用 replace 创建出两个不同的路径，例如其中一个是 `replace pkg.org1 pkg.org@xxx`，这样就能在工程中使用 `pkg.org1` 了
- 新的 module name 允许直接将版本号写在其中，如`gopkg.in/yaml.v2`，使用时可以直接用`yaml.UnMarshal()`
