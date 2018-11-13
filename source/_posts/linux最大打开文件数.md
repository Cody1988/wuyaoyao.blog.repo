---
title: linux最大打开文件数
date: 2018-11-12 20:53:37
tags:
 - linux
 - nginx
---
# nginx 访问量过大问题修复
问题如下，线上的接口大概有1000W台机器，每秒的并发量很大，导致nginx会报如下错误：

> *80789636 socket() failed (24: Too many open files) while connecting to upstream,

# 修复方案

## 修改系统最大文件访问量
系统中对于最大打开文件有两种限制，一个是`单个进程`的最大打开文件数，一个是`系统`中所有的进程可以打开的文件总数；单进程限制的配置在 `/etc/security/limits.conf ` 文件中，系统级限制在 `/etc/sysctl.conf` 

```bash
# 查看单进程最大打开文件数
> ulimit -n

1024

> ulimit -a

core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31377
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31377
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

# 查看系统最大打开文件总数量
> cat /proc/sys/fs/file-max

797699

```

此处`open files`为 1024，这个对于高并发的服务器显然太小了，需要增大。
可以在命令行零时增加为10240，查看是否还会报错

```bash
> ulimit -n 102400
```
修改后发现显著减少了一些，这里还需要修改一下`nginx`的一些链接数限制的配置: `vi /etc/nginx/nginx.conf`

```conf
worker_processes auto;
worker_rlimit_nofile 8192;
error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections  10240;
}
```
此处把 `worker_rlimit_nofile`与`worker_connections`的值均增到了很多。然后执行`service nginx reload`
至此发现没有报错误了，但是`ulimit -n 10240`命令只能零时修改最大打开文件数，当退出用户后，就失效了。需要继续下一步的配置。

## 修改配置文件`/etc/security/limits.conf`

此配置用来限制单进程的最大文件打开数，修改以下两项，并保存配置文件，这个文件正常需要重启系统才会生效，但是我们运行的服务器如果重启会很麻烦，还有一种是修改`sshd`的配置，当我们退出当前的`sshd`即可重新生效。

```conf
# * 代表对所有用户生效
# soft 代表只给出警告
 *            soft     nofile          102400
# hard 代表强制
 *            hard     nofile          102400
```
3. 修改ssh的配置文件

修改`/etc/ssh/sshd_config`，将`UsePAM`的值改为`yes`，然后重新`sshd`服务`service sshd restart`

```conf
UsePAM yes
```
`sshd`服务重启后，退出`ssh`登录，并重新链接远程shell，所有的配置即生效了。

*如果没有特别原因，设置生效后，应该把`UsePAM`的值改回去，并重启`sshd`*


查看文件打开数相关命令
---


```bash
# 查看所有进程的文件打开数
> lsof | wc -l

# 查看某个进程打开的文件数
> lsof -p pid | wc -l

# 查看系统中各个进程分别打开了多少句柄数
> lsof -n|awk ''|sort|uniq -c|sort -nr|more

```














