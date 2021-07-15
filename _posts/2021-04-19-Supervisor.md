---
layout: post
title: "Supervisor 基本功能"
date: 2021-04-19 23:00:00 +0800
tags: Server
---

![supervisor](/assets/images/2021-04-19-Supervisor_1.jpg)
一般在 linux 后台运行程序时通过`nohup ... &`实现，但是有很多不稳定的问题。
Supervisor 提供了非常方便的进程管理功能，能够满足正式线上环境的使用。

# 安装方法

`yum install -y supervisor`

安装好后，supervisor 自身的配置文件在`/etc/supervisor.conf`文件中，一般不用动

supervisor 可控制的进程关联配置文件在`/etc/supervisor.d`文件夹中比如叫`xxx.conf`

启动 supervisor 的命令：`supervisord -c /etc/supervisord.conf`

# 进程配置项

进程配置文件示例内容：

```conf
[program:test]
directory = /folder/prefix-%(program_name)s ; 进程执行目录
command = /folder/prefix-%(program_name)s/bin/%(program_name)s -c /folder/prefix-%(program_name)s/conf/config.toml ; 运行命令
autostart = true                            ; 是否随supervisord自启动
startsecs = 5                               ; 启动 5 秒后没有异常退出，就当作正常启动
autorestart = false                          ; 开启程序异常退出后自动重启
startretries = 3                            ; 启动失败自动重试次数，默认是 3
user = root                                 ; 使用root用户启动

redirect_stderr = true                      ; 把 stderr 重定向到 stdout，默认 false

stdout_logfile = /folder/prefix-%(program_name)s/log/%(program_name)s-stdout.log ; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile_maxbytes = 100MB              ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 10                 ; stdout 日志文件备份数
stdout_capture_maxbytes = 1MB               ; number of bytes in 'capturemode' (default 0)

stderr_logfile = /folder/prefix-%(program_name)s/log/%(program_name)s-stderr.log
stderr_logfile_maxbytes = 1MB
stderr_logfile_backups = 10
stderr_capture_maxbytes = 1MB
```

- 注意，如果在测试环境，为了避免程序出错被不断拉起，可能导致忽略崩溃错误，需要设置`autorestart = false`

# 常用命令

- `supervisorctl`
  打开 terminal 界面，显示各个服务状态

  - `status/reload/start/stop/restart xxx`
    对进程进行各种操作：显示状态/重新加载配置/启动/停止/重启

- `supervisorctl restart xxx`
  也可以在命令中直接输入二级命令

- supervisor 操作日志路径：
  `/var/log/supervisor/`
