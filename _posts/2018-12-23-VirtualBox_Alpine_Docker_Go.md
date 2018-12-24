---
layout: post
title:  "VritualBox+Alpine+Docker+Go 环境搭建"
date:   2018-12-23 09:00:00 +0800
tags: VirtualBox Docker
---
本文只记录了Docker环境搭建的关键步骤，适合对各个部分都很熟练的人，不适合新手。

### 平台选择
平时一直在Windows上使用VirtualBox进行Linux开发，同时有一些服务会跑在Linux虚拟机里。

现在Docker性能这么优秀，要尽快学习，日常利用跑一些服务也更节省资源。

Alpine是一种精简、安全的Linux操作系统，系统镜像文件只有104M。因为我们只想运行Docker，所以选择Alpine作为宿主系统。

### 安装VirtualBox
忽略安装过程，要安装最新版本[下载地址](https://www.virtualbox.org/wiki/Downloads)，并且扩展版本要与虚拟机版本一致。

### 安装Alpine、Docker
下载地址[下载地址](https://alpinelinux.org/downloads/)，下载标准版镜像。

※由于Alpine相对小众，所以发生问题先到官网找一下方法，不同版本可能处理方式不同，乱搜不容易找到答案。[官网VirtualBox安装说明](https://wiki.alpinelinux.org/wiki/Install_Alpine_on_VirtualBox)

在VirtualBox创建虚拟机，类型选择"Linux 2.6 / 3.x / 4.x (64-bit)"，创建动态扩展硬盘，设置下面选项：

开启PAE/NX

开启硬件加速

网卡配置成"桥接网卡"(本地路由器DHCP要好用)

USB设置3.0

共享文件夹添加一个路径：C:\test,名称设置"test_folder"

光驱选择iso文件，设置刚才下载的镜像

启动虚拟机，默认从光盘启动，默认一路回车，只记录关键点：

root登录，没有密码

setup-alpine		// 开始安装

为root设置密码时，输入合适的密码

选择时区时，输入"Asia/Shanghai"

安装硬盘时，输入"sda"

选择分区时，输入"sys"

安装完毕,poweroff,关机

在VirtualBox卸载光盘

再次启动虚拟机

Alpine默认root用户不可以SSH登录，为了方便使用，编辑/etc/ssh/sshd_config，加入一行"PermitRootLogin yes"。

由于VirtualBox、Docker等需要用到community仓库，所以要把"/etc/apk/repositories"文件中两个community前面的注释删掉。

apk add virtualbox-guest-additions virtualbox-guest-modules-virt	// 安装VirtualBox增强扩展，执行命令

reboot	// 重启

用root登录

mkdir test_folder	// 准备好可以映射到外部的文件夹

modprobe -a vboxsf	// 加载VirtualBox增强模块

mount -t vboxsf test_folder /root/test_folder	// 手动加载映射，之后测试一下

手动测试没有问题后，将自动加载代码"test_folder /root/test_folder vboxsf defaults 0 0"加入文件"/etc/fstab"最后一行。

apk add docker	// 安装Docker

ln -s /etc/init.d/docker /etc/runlevels/default/docker	// 添加Docker服务自启动

mkdir /etc/docker		// 创建好配置用文件夹

创建文件"/etc/docker/daemon.json"，添加国内镜像加速：
```
{
"registry-mirrors": ["https://registry.docker-cn.com"]
}
```

重启，Docker安装完成

### 测试Docker
执行"docker run hello-world"，将会自动下载hello-world image，然后创建一个容器运行image，打印出一系列文字：
```
alpine-host:~# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:41a65640635299bab090f783209c1e3a3f11934cf7756b09cb2f1e02147c6ed8
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

按照提示，又执行了"docker run -it ubuntu bash"，果然进入了bash，可以执行各种基本命令。进入一个系统不到一秒！

### 创建Go HelloWord容器
在Windows"C:\test"下创建Go测试代码，main.go：
```
package main
import (
  "fmt"
  "net/http"
)

const (
  host string = ":80"
)

func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello world!")
  })

  fmt.Println("Listening ", host)
  http.ListenAndServe(host, nil)
}
```

下载最新的Alpine镜像到本地
```
docker pull alpine
```

创建数据卷容器
```
docker run -it --name=test_folder1 -v /root/test_folder:/test_folder2:Z alpine:latest
```
* 其中，test_folder1是容器的名字，后面会用到
* -it 是终端登录模式，之后可以查看文件用
* -v 是从宿主加载文件夹或文件到容器中
* "root/test_folder"是我们创建虚拟机时加载的共享文件夹
* "test_folder2"是容器中的对应路径
* ：Z是有些操作系统(如CentOS)会用到，加上没有坏处

下载最新的Go官方提供的Alpine镜像
```
docker pull golang:alpine
```

以数据卷容器为卷容器启动go:alpine，编译、运行我们在共享文件夹创建的"go.main"，绑定容器和宿主的80端口
```
docker run -it -p 80:80 --volumes-from test_folder1 golang:alpine go run /test_folder2/main.go
```

虚拟机屏幕显示"Listening :80"，在Windows下打开Chrome，输入虚拟机地址"xxx.xxx.xxx.xxx"回车，就可以看到"Hello world!"了。

※上面的操作中，使用了latest作为container的tag，实际上这种做法是不好的，不利于版本控制，最好要指定版本。





