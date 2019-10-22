---
layout: post
title: "Linux Shell 常用命令"
date: 2018-09-02 10:00:00 +0800
tags: Linux
---

## 进程相关操作

`Ctrl+C`
关闭程序(发送 SIGINT)

`Ctrl+Z`
暂停进程(发送 SIGTSTP)

`Ctrl+D`
输入 EOF

`jobs`
查看已经暂定的进程

`fg %###`
将后台进程加载到前台运行

`bg %###`
在后台继续运行暂停的进程。如果要退出 terminal 时仍然运行后台进程，进程的输出要重定向，例如 xxx >>tmp.log

`xxx \<回车>`
可以实现多行命令一起执行

`xxx1 ; xxx2 ; xxx3`
顺序执行多条命令。

`xxx1 | xxx2 | xxx3`
`|`piepline(管道)，表示左边(前边)执行的命令结果作为右边命令的参数传入，可以连续使用。

`xxx1 & xxx2 & xxx3`
`&`符号左边的命令后台执行(多进程)，

`xxx1 || xxx2`
或关系，表示上一条命令执行失败后才执行下一条命令。

`xxx1 && xxx2 ; wait`
与关系，表示上一条命令执行成功后才执行下一条命令。一般最后要加一个`wait`，让 shell 主进程等待子进程执行完毕再退出，避免主进程过早结束导致子进程意外结束。

### 显示相关

`history`
查看历史命令

`clear`
清空屏幕显示

`xxx|more`
通过命令行滚动查看执行结果。b 向前翻页，空格向后翻页

## 系统信息

`whoami`
查看当前使用的用户名

`uname -a`
查看操作系统信息

`cat /etc/issue`
查看 CentOS 系统版本

切换为 root 用户权限
su // 相当于 su root
su - root // 切换时，同时保持环境变量
在"/etc/passwd"中可以查看已经存在的用户及用户组

改变文件权限为可执行可修改
chmod 755 ./xxx

更新一下文件的最新修改时间
touch ./xxx

将某文件或文件夹授权给某用户
chown username filefoldername

insmod
将文件安装到内核
通常是驱动。

ls 列出文件名
ls -a 列出所有文件，包含隐藏文件
ls -l 列出详细信息，最后会表示软连接信息

ll
列出详细信息
显示的时间为文件或文件夹的修改时间

du --apparent-size
显示当前文件夹空间以及各子文件夹空间

pwd
当前目录

反向显示列表内容
ll -t | tac

以分页形式查看命令结果
xxx |less

当前路径打开终端
ctrl+shift+t

复制 ctrl+insert
粘贴 shift+insert

查询命令帮助
man xxx

当前文件夹正则搜索
find . -name "xxx"

全盘定位文件
locate xxx
locate -r xxx 正则查找

查找特定时间之后创建或修改的文件
touch -t 201407201710.00 abc //在当前文件夹创建一个 2014 年 07 月 20 日 17 点 10 分 00 秒 创建的临时文件
find -newer abc // 在当前文件夹下，查找比刚才那个临时文件更新的文件

`sudo updatedb`
更新 locate 数据库

`grep "test" -r /usr`
按文件内容搜索

`cat xxx`
查看文件文本内容

连续执行命令
xxx;xxx

管道，前面命令结果作为后面命令参数
xxxx | xxxx

whereis xxx
查找命令路径

which xxx
返回路径，同时可以判断 xxx 的类型，是 bin、函数还是 gems 等

cat xxx | grep error
查找某文件中包含 error 字符串的行

cat /etc/issue
查看操作系统版本

ifconfig
查看 IP 信息

wall
广播一段文字消息，以 Ctrl+D 输入 EOF 结束符。

创建一个软连接，相当于快捷方式
ln -s /sourcedirectory /destdirectory
source 是已经存在的文件或文件夹
dest 是将要创建的文件或文件夹

查看进程 PS(Process Shot)
ps -ef
查看进程位置
ps -ef|grep xxxx

查看进程内存使用
ps -aux

查看进程端口使用情况
netstat -ntlp

查看<设定>最大连接数
ulimit -n <newValue>

内存总容量查询
free

运行某个".sh"文件
bash file.sh
sh file.sh
sh -x file.sh // 在执行命令时显示执行细节
./file.sh

查看磁盘剩余空间(Disk File)
df -h

du -sh \*
查看当前文件夹下各文件和文件夹的容量

查看 CPU 使用情况
top

把执行结果重定向到文件
cmd > file // 覆盖原有文件
cmd >> file // 追加到原文件后面
cmd 1> file // 只重定向 stdout
cmd 2> file // 只重定向 stderr
cmd &> /dev/null // 抛弃所有的 log

\$? -eq 0 // 判断前面执行的命令是否成功，一般成功返回 0，失败返回 1

修改密码
passwd //修改当前用户密码
passwd name //修改 name 用户的密码

dpkg -i xxx.deb
安装.deb 包。

apt-get update //更新最新的软件版本列表
apt-get upgrade //将所有软件更新到最新版本，系统也可能升级
apt-get install xxx //安装某个软件

压缩文件夹
tar -zcvf /home/xahot.tar.gz /xahot --exclude=xxx
tar -zcvf 打包后生成的文件名全路径 要打包的目录 要排除的目录名
tar –xvf file.tar 解压 tar 包
tar -xzvf file.tar.gz 解压 tar.gz
tar -xjvf file.tar.bz2 解压 tar.bz2
tar –xZvf file.tar.Z 解压 tar.Z

poweroff 关机

date 查看系统时间
clock --show 查看硬件时间
clock --hctosys 将硬件时间应用到系统时间

cp -Rf /home/user1/\* /root/temp/
将 /home/user1 目录下的所有东西拷到/root/temp/下而不拷贝 user1 目录本身。
即格式为：cp -Rf 原路径/ 目的路径/

last -10
查看最后 10 次登录情况

iptables -nv -L
查看网络访问控制情况

`kill [-signalSeq] processId`
向进程发送信号。在不设定信号序号时默认发送`SIGTERM`。

`kill -l`
显示可发送信号列表。

`export VALUE_NAME="value"`
设置环境变量

`echo $VALUE_NAME`
输出环境变量的值

`unset VALUE_NAME`
删除环境变量
