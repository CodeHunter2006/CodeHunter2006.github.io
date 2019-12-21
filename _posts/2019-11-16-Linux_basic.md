---
layout: post
title: "Linux基础"
date: 2019-11-16 20:00:00 +0800
tags: Linux
---

![Vim](/assets/images/2019-11-16-Linux_basic_1.jpg)
记录 Linux 基本概念、结构和用法

# Kernel, Shell, Bash

通常我们说的 Linux 其实就是"Linux Kernel(内核)"的简称，它提供了 Linux 最核心的功能。而`Shell`直译为外壳，它基于 Linux Kernel，为用户提供操作 Linux 的各种命令接口或者为程序提供函数接口。`Bash`是 Linux 自带的一个命令行工具软件，提供命令行界面来执行`Shell`命令或其它可可执行文件。

# 文件

## hard link(硬连接), symbolic link(符号连接/软连接), alias(mac)

每个 linux 文件实体都会绑定一个唯一的 Inode Index(节点索引)，并且会默认分配一个 hard link，这个 hard link 就是我们通常看到的一个"文件"，结构如下：
可以看到的"文件"(hard link/symbolic link/alias) -> Inode Index(文件实体)

- hard link 会直接指向文件，对 hard link 的操作会直接影响文件本身。可能存在多个 hard link 指向同一个 Inode Index 的情况，这时删除其中一个 hard link 并不会删除文件，当所有 hard link 被删除时会真正删掉文件。
- symbolic link 是一个独立的文件，其中存储着一个特定的文件路径。删除 symbolic link 不会对实体文件或指向实体文件的 hard link 有任何影响。而删除或移动文件后，指向它的 symbolic link 会失效。
- alias 是 mac 系统提供的一种文件，类似 symbolic link 也是一个独立的文件，相比 symbolic link 增加了 Inode Index 信息，可以自动追踪被移动的文件，不会因为文件移动位置而失效。alias 只在 mac 系统下起作用。
- 使用`ls -li`命令可以查看文件的 Inode Index 和软连接信息。注意 alias 在 linux 中只是一个普通的文件，并不会认为有连接关系。
  - 执行命令时可以看到容量关系 `hard link` > `alias` > `symbolic link`，与前面的说明一致，`hard link`直接显示文件本身的容量，`alias`比`symbolic link`包含的信息更多所以容量更大。

# 常用服务

## cron

cron 是 Linux 自带的 service 提供定时执行命令的功能，类似 Windows 任务计划。

- cron 可执行文件位于`/usr/sbin/crond`

- 可以用下面语句对 cron 进行服务操作
  /sbin/service crond start //启动服务
  /sbin/service crond stop //关闭服务
  /sbin/service crond restart //重启服务
  /sbin/service crond reload //重新载入配置

- 在/etc/rc.d/rc.local 这个脚本的末尾加上：
  `/sbin/service crond start`
  则系统启动时就会运行 cron 服务
