---
layout: post
title: "Linux Shell 常用命令"
date: 2018-09-02 10:00:00 +0800
tags: Linux
---

# 命令

## 进程相关操作

`Ctrl+C`
关闭程序(发送 SIGINT)

`Ctrl+Z`
暂停进程(发送 SIGTSTP)

`Ctrl+D`
输入 EOF

`jobs`
查看后台(运行/暂停)的进程

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

## 显示相关

`history [n]`
查看历史命令

- `[n]` number，打印最近的 n 条命令
- `-c` clear，清空当前历史命令缓冲区
- `-w` write，将当前历史命令缓冲区写入文件。如果当前命令缓冲区为空则删除文件中所有历史命令
- `-r` read，将文件中历史命令读入当前历史命令缓冲区
- `-a` append，将当前历史命令缓冲区续写入文件
- `CTRL+R` 进入搜索模式，输入部分命令可以在历史中搜索

```bash
export HISTSIZE=1000         # 设置内存中的history命令的个数
export HISTFILESIZE=1000     # 设置文件中的history命令的个数
```

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

## 权限控制

`chmod +x xxx/xxx`
赋予文件可执行权限

`chmod 755 ./xxx`
改变文件权限为可执行可修改

`chown username filefoldername`
将某文件或文件夹授权给某用户

`sudo xxx`
以 root 权限执行命令

- `-u` 以指定的身份(默认 root)作为新的用户身份，保持 root 权限执行以后的命令。

更新一下文件的最新修改时间
touch ./xxx

insmod
将文件安装到内核
通常是驱动。

## ls

列出文件名

- `-a` 列出所有文件，包含以`.`开头的隐藏文件
- `-l` 以列的方式显示，列出详细信息(权限、修改时间、软连接信息)
- `-i` 显示 Inode Index 序列号

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

## find

查找文件

`find root_path -name "abc*"`
在 root_path 下按名字查找文件，其中`*`为通配符

- `-newer xyz` 查找比 xyz 文件修改时间更新的文件

## touch

更新文件时间

`touch xxx`
更新 xxx 的修改时间为当前时间

- `-t 201407201710.00` 更新到一个特定时间，如`2014 年 07 月 20 日 17 点 10 分 00 秒`

全盘定位文件
locate xxx
locate -r xxx 正则查找

`sudo updatedb`
更新 locate 数据库

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

## nohup

no hang up(不挂起)，将当前命令的输出改为当前目录的 nohup.out 文件，避免在终端退出时程序中断

`nohup Command [ Arg … ] [ & ]`
在后台执行某个命令，重定向输出到 nohup.out

## ifconfig

查看 IP 信息
`ifconfig en0`
查看本地 IP

wall
广播一段文字消息，以 Ctrl+D 输入 EOF 结束符。

## ln

link 创建一个文件或文件夹的链接，这样可以在修改一处时影响多处。

`ln source target`
创建一个硬链接，相当于不同文件名对应同一个文件。

- `-s` 创建软连接，相当于快捷方式

查看进程 PS(Process Shot)
ps -ef
查看特定进程
ps -ef|grep xxxx

查看进程内存使用
ps -aux

查看进程端口使用情况
netstat -ntlp

`lsof -i:8090`
查看占用特定端口的进程

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

- `-d num`设置刷新间隔，单位秒
- `-n num`设置刷新采样 num 次后退出

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
输出(环境)变量的值

`unset VALUE_NAME`
删除环境变量

# 命令相关操作

## history

查看历史命令。

- 这些命令都被写到了`~/.bash_history`中，可以用环境变量`HISTSIZE`设置保存的命令数。
- 按'上/下'按键可以查看历史命令
- `CTRL+R`可以进入命令搜索状态，可以搜索历史命令

# 文本操作

## grep

global regular expression print
在文本、文件中查找符合模式的文本行。

`cat aaa | grep bbb`
在 cat 命令结果的字符串中查找 bbb

`grep -e "test.*" xxx.xx`
在 xxx.xx 文件中查匹配`test.*`正则表达式的行

- `-v` invert-match，查找没有出现目标文本的内容
- `-o` --only-matching，只输出完全匹配模式的部分
- `-A n` add，增加匹配行后 n 行的显示
- `-B n` before，增加匹配行前 n 行的显示
- `-C n` context，增加匹配行前后 n 行的显示
- `-c` count，只输出匹配的总行数
- `-e` regexp，匹配正则表达式
- `-m` max-count，控制输出的最大行数
- `-n` 同时输出行号
- `-d` directory，以文件夹为目标进行搜索
- `-r` recursive，递归搜索自文件夹
- `-l` 只列出匹配的文件，不显示具体行
- `--color=auto` 搜索到的文字高亮显示

正则表达式

- `[0-9]` 0-9 的数字
- `[0-9]*` 0-9 的数字出现 0 次到多次
- `.*` 任意字符出现多次
- `\.` . 出现一次
- `[a-z]` a-z 的小写字母
- `[a-zA-Z0-9]` 多种分段的组合
- `[123]` 匹配 1 或 2 或 3，只能匹配一个字符
- `[^1]` 反向选择，匹配没有 1 的
- `[^a-zA-Z]` 反向选择，没有字母的
- `"^test"` 定位行首为 test 的行
- `"test$"` 定位行尾为 test 的行

## tail

从文件末尾开始对日志保持观察

`tail -20f xxx.log`
从最后 20 行开始观察文件

## tailf

与 tail 功能相同，会从文件开头读起，并对末尾保持观察。

- 对于较大的文件 tailf 开始时候会较慢，因为 tail 会从末尾反向读取
- tailf 对文件没有增长的文件不会去读取，所以会较省电一点

`tailf -20 xxx.log`
从最后 20 行开始观察文件

## less

以较小的缓存区读取文件，不需要全部扫描文件后再打开问文件，适合查看容量较大的 log。less 的最小操作单位和 grep 一样，以行为单位。

`less xxx/xxx`
打开文件查看

## diff

查看文件差异

`diff file1 file2`
显示两个文件的差异

# 远程通信

## ssh

## scp

基于 ssh 协议传输文件，会要求输入密码

`scp [-r] source target`
从源拷贝到目标

- 本地路径可以用绝对或相对路径表示
- 目标路径用`username@servername:/path`表示
- `-r` 表示以文件夹形式传输

# shell 语法

## 注释

```shell
# 注释在 '#' 右边
```

## 变量赋值

`VALUE_NAME=value`
给变量赋值，右边直接写值，字符串也可以直接写

`VALUE_NAME=(grep test testFile)`
将命令执行结果作为值赋值给变量

```shell
VALUE_NAME1=value1
VALUE_NAME2=value2

# 用一个变量的值赋值给另一个变量
# VALUE_NAME3 的结果是 "value1"
VALUE_NAME3=$VALUE_NAME1

# 用变量的值进行字符串拼接，结果赋值给另一个变量
# VALUE_NAME4 的结果是 "value1+value2"
VALUE_NAME4=${VALUE_NAME1}+${VALUE_NAME2}
```

## 字符串

shell 中的字符串与脚本命令之间不像 C++那样"隔绝"，往往可以自由的切换使用，所以需要搞清什么时候是字符串、什么时候是执行的命令。

## 调用参数

在命令行调用脚本文件时，可以在后面传参，例如`./my.sh p1 p2`，然后在脚本中可以用一些符号表示参数。

`$#`
表示参数个数

`$0`
被执行的脚本本身的文件路径

`$1`
传入的第一个参数。`$2`、`$3`...表示后面的参数

## 条件判断

如果文件存在，则删除该文件。

```shell
FILE_PATH=/test/test.txt
if [ -f "$FILE_PATH" ]; then
    rm -rf $FILE_PATH
fi
```

# 应用场景

## 查找特定时间之后创建或修改的文件

```shell
touch -t 201407201710.00 abc
# 在当前文件夹创建一个 2014 年 07 月 20 日 17 点 10 分 00 秒 创建的临时文件

find -newer abc
# 在当前文件夹下，查找比刚才那个临时文件更新的文件
```
