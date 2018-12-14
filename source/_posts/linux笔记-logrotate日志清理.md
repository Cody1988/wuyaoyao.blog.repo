---
title: linux笔记-logrotate日志清理
date: 2018-12-14 14:21:54
tags:
    - linux
---

# 需求
服务器上面，日志太大，导致一些写入的接口无法调用，经过排查发现时服务器没有安装 `crontab` 导致 `logrotate` 没有定时运营

# 安装 crontab

```bash
# 安装crontab
> yum install crontab
```

# logrotate

`logrotate` 程序是一个日志文件管理工具。它可以自动对日志进行截断（或轮循）、压缩以及删除旧的日志文件。可以节省磁盘空间。它的配置文件主要包含： `/etc/logrotate.conf`、`/etc/logrotate.d` 两个配置的方式，其中 `/etc/logrotate.conf` 会导入 `/etc/logrotate.d` 目录下的所有的配置文件。

`logrotate` 基于 `cron` 运行，脚本在 `/etc/cron.daily/logrotate` 。

## 配置

通常服务器端需要对运行的服务进行日志定期的截取，比如 nginx、tomcat、mysql的日志。这是可以到 `/etc/logrotate.d` 目录下进行配置。

```bash
# 生成一个10M的随机数据的文件
> touch /var/log/test
> head -c 10M < /dev/urandom > /var/log/test

# 创建logrotate配置文件，并编辑
> vi /etc/logrotate.d/test
/var/log/test {
  # 日志按照天轮询 daily,weekly,monthly,yearly
  daily
  # 保存5个日志，多于5个的将时间最久的删除
  rotate 5
  # 日志以日期为后缀
  dateext
  # 日志gzip压缩
  compress
  # 延迟压缩
  delaycompress
  # 忽略错误
  missingok
  # 日志为空则不轮询
  notifempty
  以指定的权限创建新的日志文件
  create 666 root root
  # 完成其它命令后执行的脚本
  postrotate
    /usr/bin/killall -HUP rsyslogd
  endscript
}
```

## 手动运行 logrotate

```bash
# 运行test
> logrotate /etc/logrotate.conf/test
# debug 运行
> logrotate -d /etc/logrotate.conf/test
# 强制运行
> logrotate -vf /etc/logrotate.conf/test
```

## logrotate 定时任务

logrotate 定时执行需要使用 `crontab`

```bash
> cat /etc/cron.daily/logrotate 
#!/bin/sh

/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```


