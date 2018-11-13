---
title: mysql安装与迁移
date: 2018-11-12 20:51:07
tags:
  - 服务器
  - mysql
---

# centos7 安装mysql

1. 到 `http://repo.mysql.com/` 下载对应的 `rpm` 文件；
2. 执行命令行 `rpm -ivh mysql-community-release-xxx.noarch.rpm`;
3. 安装 `mysql`：`yum install mysql-server`

# 设置密码

```bash
# 
> mysql -u root
# 选择mysql数据库
mysql> user mysql
# 修改密码
mysql> set password for 'root'@'localhost' = password('mypasswd'); 

```

# mysqld

```bash
# 启动
> service mysqld start
# 停止
> service mysqld stop
# 运行状态
> service mysqld status
# 重启
> service mysqld restart
```

# 进入mysql

```bash
# 进入mysql并use 数据库
> mysql -uroot -p'abc.123'  --default-character-set=utf8 xxxdb
```

# 导出数据

```bash
> mysqldump -uroot -p'abc.123'  --default-character-set=utf8 xxxdb > xxxxdb.sql

```

# mysql 存储位置修改

`CentOS` 中 `mysql` 数据库默认的存储位置为 `/var/lib/mysql` 通常系统盘比较小，我们需要把数据库的位置移到数据盘中比如 `/data/mysql` 目录。首先停止数据库服务 `service mysqld stop`。

## 拷贝数据目录

```bash
# 完整拷贝目录，包括操作权限
> cp -arp /var/lib/mysql /data/

```

## 修改配置

`/etc/my.cnf`

```conf
# datadir=/var/lib/mysql
datadir=/data/mysql
# socket=/var/lib/mysql/mysql.sock
socket=/data/mysql/mysql.sock
```

`/etc/init.d/mysqld`

```conf

# get_mysql_option datadir "/var/lib/mysql" mysqld
# 修改默认的数据存储路径
get_mysql_option datadir "/data/mysql" mysqld

```

```conf
# DATADIR=/var/lib/mysql
# wuyaoyao 修改默认的存储路径
DATADIR=/data/mysql
```

至此配置已经修改完毕了，可以重新启动数据库

```bash
# 数据库启动成功，但是会有 waning 信息，并且有提示需要重新加载单元信息
> service mysqld start

Starting mysqld (via systemctl):  Warning: mysqld.service changed on disk. Run 'systemctl daemon-reload' to reload units.

> systemctl daemon-reload

```

此时运行命令行链接数据库

```bash
> mysql -u root
ln: 无法创建符号链接"/var/lib/mysql/mysql.sock": 没有那个文件或目录

```

那么需要创建一个软连接

```bash
> ln -s /data/mysql/mysql.sock /var/lib/mysql/mysql.sock
```

至此数据就迁移完成了