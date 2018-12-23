---
layout: post
title:  "VritualBox+Alpine+Docker+Go 环境搭建"
date:   2018-12-08 10:00:00 +0800
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
* 开启PAE/NX
* 开启硬件加速
* 网卡配置成"桥接网卡"(本地路由器DHCP要好用)
* USB设置3.0
* 共享文件夹添加一个路径：C:\test,名称设置"test_folder"
* 光驱选择iso文件，设置刚才下载的镜像

启动虚拟机，默认从光盘启动，默认一路回车，只记录关键点：
* root登录，没有密码
* setup-alpine		// 开始安装
* 为root设置密码时，输入合适的密码
* 选择时区时，输入"Asia/Shanghai"
* 安装硬盘时，输入"sda"
* 选择分区时，输入"sys"
* 安装完毕,poweroff,关机
* 在VirtualBox卸载光盘
* 再次启动虚拟机
* Alpine默认root用户不可以SSH登录，为了方便使用，编辑/etc/ssh/sshd_config，加入一行"PermitRootLogin yes"。
* 由于VirtualBox、Docker等需要用到community仓库，所以要把"/etc/apk/repositories"文件中两个community前面的注释删掉。
* apk add virtualbox-guest-additions virtualbox-guest-modules-virt	// 安装VirtualBox增强扩展，执行命令
* reboot	// 重启
* 用root登录
* mkdir test_folder	// 准备好可以映射到外部的文件夹
* modprobe -a vboxsf	// 加载VirtualBox增强模块
* mount -t vboxsf test_folder /root/test_folder	// 手动加载映射，之后测试一下
* 手动测试没有问题后，将自动加载代码"test_folder /root/test_folder vboxsf defaults 0 0"加入文件"/etc/fstab"最后一行。
* apk add docker	// 安装Docker
* ln -s /etc/init.d/docker /etc/runlevels/default/docker	// 添加Docker服务自启动
* mkdir /etc/docker		// 创建好配置用文件夹
* 创建文件"/etc/docker/daemon.json"，添加国内镜像加速：
```
{
"registry-mirrors": ["https://registry.docker-cn.com"]
}
```
* 重启，Docker安装完成

### 创建HelloWord容器






