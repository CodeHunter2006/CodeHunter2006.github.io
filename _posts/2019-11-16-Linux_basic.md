---
layout: post
title: "Linux基础"
date: 2019-11-16 20:00:00 +0800
tags: Linux
---

![Vim](/assets/images/2019-11-16-Linux_basic_1.jpg)
记录 Linux 基本概念、结构和用法

# Kernel, Shell, Bash

通常我们说的 Linux 其实就是"Linux Kernel(内核)"的简称，它提供了 Linux 最核心的功能。而`Shell`直译为外壳，它基于 Linux Kernel，为用户提供操作 Linux 的各种命令接口或者为程序提供函数接口。`Bash`是 Linux 自带的一个终端(terminal)命令行工具软件，提供命令行界面来执行`Shell`命令或其它可可执行文件。

# 文件

## hard link(硬连接), symbolic link(符号连接/软连接), alias(mac)

每个 linux 文件实体都会绑定一个唯一的 Inode Index(节点索引)，并且会默认分配一个 hard link，这个 hard link 就是我们通常看到的一个"文件"，结构如下：
可以看到的"文件"(hard link/symbolic link/alias) -> Inode Index(文件实体)

- hard link 会直接指向文件，对 hard link 的操作会直接影响文件本身。可能存在多个 hard link 指向同一个 Inode Index 的情况，这时删除其中一个 hard link 并不会删除文件，当所有 hard link 被删除时会真正删掉文件。
- symbolic link 是一个独立的文件，其中存储着一个特定的文件路径。删除 symbolic link 不会对实体文件或指向实体文件的 hard link 有任何影响。而删除或移动文件后，指向它的 symbolic link 会失效。
- alias 是 mac 系统提供的一种文件，类似 symbolic link 也是一个独立的文件，相比 symbolic link 增加了 Inode Index 信息，可以自动追踪被移动的文件，不会因为文件移动位置而失效。alias 只在 mac 系统下起作用。
- 使用`ls -li`命令可以查看文件的 Inode Index 和软连接信息。注意 alias 在 linux 中只是一个普通的文件，并不会认为有连接关系。
- 执行命令时可以看到容量关系 `hard link` > `alias` > `symbolic link`，与前面的说明一致，`hard link`直接显示文件本身的容量，`alias`比`symbolic link`包含的信息更多所以容量更大。

- 软链接：
  1. 软链接是存放另一个文件的路径的形式存在。
  2. 软链接可以跨文件系统 ，硬链接不可以。
  3. 软链接可以对一个不存在的文件名进行链接，硬链接必须要有源文件。
  4. 软链接可以对目录进行链接。
- 硬链接：
  1. 硬链接，以文件副本的形式存在。但不占用实际空间。
  2. 不允许给目录创建硬链接。
  3. 硬链接只有在同一个文件系统中才能创建。
  4. 删除其中一个硬链接文件并不影响其他有相同 inode 号的文件。

* 硬链接的作用：
  1. 节省硬盘空间。bai 同样的文件，只需要 du 维护硬连接关系，不需要进行多重的 zhi 拷贝，这样可以节省硬盘空间。
  2. 重命名文件。重命名文件并不需要打开该文件，只需改动某个目录项的内容即可。
  3. 删除文件。删除文件只需将相应的目录项删除，该文件的链接数减 1,如果删除目录项后该文件的链接数为零，这时系统才把真正的文件从磁盘上删除。
  4. 文件更新。如果涉及文件更新，只需要先在 WinSxS 目录里面下载好一个新版本，然后修改面同名文件的硬即可改变；

一个文件包含两个参数`i_count`和`i_link`。i_count 表示被调用的数量；i_link 表示被介质连接的数量(硬连接)。当它们同时为 0 时，会被系统删除。

- 例如：用`rm`命令删除的文件，i_link 减 1，如果此时这个文件还在被某个进程使用，则 i_count 不为 0，该文件还可以继续被该进程使用，直到进程释放资源后才会被删除。

# 文件夹结构

`/tmp/`
该文件夹下可以存放临时文件，Debian/Ubuntu 每次启动会清空一次，RHEL/CentOS/Fedora 每隔特定天数(至少一天)会清空一次近期未更新的文件。

`/var/log/`
系统日志目录

# 常用服务

## crontab

cron 是 Linux 自带的 service 提供定时执行命令的功能，类似 Windows 任务计划。

`crontab -e`
打开 cron 的定时任务编辑页面，可参照说明进行编辑。
编辑完成退出后显示`crontab：installing new crontab`表示成功添加任务，不过这个任务要在三分钟后才生效，所以时间过近的不会执行。

- `-l` 显示已有的配置列表

```sh
分 时 日 月 周 命令
#第一列代表分钟1~59：为*表示每分钟都要执行；为*/n表示每n分钟执行一次；为a-b表示从第a分钟到第b分钟这段时间每分钟要执行；a-b/c表示从a-b这段时间每c分钟执行一次；为a,b,c,...表示第a,b,c分钟要执行
```

- cron 可执行文件位于`/usr/sbin/crond`
- 不同的用户配置不同，root 用户用`sudo crontab -e`进入配置

- 可以用下面语句对 cron 进行服务操作
  /sbin/service crond start //启动服务
  /sbin/service crond stop //关闭服务
  /sbin/service crond restart //重启服务
  /sbin/service crond reload //重新载入配置

- 在/etc/rc.d/rc.local 这个脚本的末尾加上：
  `/sbin/service crond start`
  则系统启动时就会运行 cron 服务

# 内核机制

## OOM killer(Out Of Memory killer)

当系统内存不足时，会杀掉占用内存最大的程序，并输出系统日志。
查看相关日志的命令`grep "Out of memory" /var/log/messages`、`egrep -i -r 'killed process' /var/log`

# 进程

在 Linux 操作系统中，一个进程的内存空间被分为两部分：内核堆栈、用户堆栈，对应的进程有两个权限状态：内核态、用户态。
内核态为完全可信的内核代码，可以执行高风险的 CPU 指令(0 等级)，可以与硬件交互，可以操作内核堆栈和用户堆栈；
用户态为程序自己的代码，可以执行低风险的 CPU 指令(3 等级)，不可以直接与硬件交互，可以操作用户堆栈，可以通过内核函数调用与内核态资源交互。

当用户态和内核态相互切换时，CPU 寄存器就会进行备份、切换，涉及到寄存器(示例)：代码地址、代码行号、堆栈地址、操作数 1、操作数 2...

## 进程线程

Linux 最初是没有线程的设计的，后来其他操作系统有了线程，Linux 就通过进程模拟线程，通过 fork 创建进程将新的进程的资源句柄与原进程一致，这样就实现了线程的功能。

# 内存

可以用`free`命令查看内存情况

- **used**
  已使用的内存容量，包含 cached buffers shared 三部分
- **free**
  空闲内存容量
- **cached**
  这里指内存与硬盘间的 cache，主要针对读操作设计。把频繁从硬盘读取的数据缓存起来，之后如果再访问可直接从内存读取，减少磁盘访问耗时。
- **buffers**
  把要写到硬盘的数据先缓存一下，稍后再刷到硬盘上，这样避免频繁写操作耗时。Linux 有一个守护进程定期清空缓存内容将其写入硬盘，手动执行 sync 命令也可以把数据刷到硬盘。
- **shared**
  进程间共享内存的容量
- **swap**
  交换区，相当于 Windows 的虚拟内存。当内存不足时，将硬盘用作部分内存使用，扩展内存容量。

# CPU

可以用`top`命令查看 CPU 情况

- **load**
  CPU 负载，表示排队等待 CPU 的进程数量。如果`load average`除以 CPU 数量 > 5，则表明 CPU 可能超负荷运行，需要关注。
  - CPU 负载高的可能原因:1. 正在进行 CPU 密集型计算；2. 可能 CPU 正在等待 IO(内存、硬盘、网络)；
- **us(user cpu time)**
  用户态使用 CPU 占比。
- **sy(system cpu time)**
  内核态(系统态)使用 CPU 占比。
- **wa(io wait cpu time)**
  CPU 等待 IO 的时间占比。如果该值较高，说明 IO 出现了瓶颈，需要具体分析是磁盘/内存/网络。

可以用`nice`命令来调整进程优先级。范围为-20(最高)~19(最低)，默认值为 10。

# Cache

高速缓存，位于 CPU 与内存之间的容量小速度快的存储器。根据程序的局部性原理设计，将频繁访问的数据放入 Cache 中，避免频繁读内存。Cache 分为 L1 Cache 和 L2 Cache，L1 Cache 位于 CPU 内部，速度更快容量更小，L2 Cache 以前位于主板，后来也直接封装到 CPU 中。

- Cache 是一种设计思想，可以用于各种速度不匹配的存储之间，提高读取效率

# Buffer

缓冲区，用于不同速度存储之间传输数据，避免通信次数过多造成效率低下。
用于写入时，可以将要写入的数据先放入高速存储的 Buffer，然后再定时一次性写入低速存储；
用于读取时，可以将要读取的低速存储中的数据包含附近数据一次性读取高速存储的 Buffer，然后慢慢处理。

- Buffer 和 Cache 一样是一种设计思想，用于速度不匹配的存储之间，提高写入效率(有时也用于读取，比如从磁盘读取整块数据块)
