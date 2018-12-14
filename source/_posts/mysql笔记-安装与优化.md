---
title: mysql笔记-安装与优化配置
date: 2018-11-12 20:51:07
tags:
  - 服务器
  - mysql
---
# centos7 安装mysql

1. 到 `http://repo.mysql.com/` 下载对应的 `rpm` 文件；
2. 执行命令行 `rpm -ivh mysql-community-release-xxx.noarch.rpm`;
3. 安装 `mysql`：`yum install mysql-comminuty-server`

# 设置 mysqld 开机启动

```bash
# 添加开机启动配置
> chkconfig -add mysqld
# 列出 mysql 开机启动配置
> chkconfig --list mysqld
mysqld          0:关    1:关    2:关    3:开    4:开    5:开    6:关

```

# 设置密码

```bash
# 查找mysql密码，此处初始密码为：0w>jfdwu-qLe
> grep password /var/log/mysqld.log
2018-11-29T08:59:40.033885Z 1 [Note] A temporary password is generated for root@localhost: 0w>jfdwu-qLe
2018-11-29T08:59:43.218343Z 0 [Note] Execution of init_file '/var/lib/mysql/install-validate-password-plugin.v6UOBy.sql' started.
2018-11-29T08:59:43.220319Z 0 [Note] Execution of init_file '/var/lib/mysql/install-validate-password-plugin.v6UOBy.sql' ended.
2018-11-29T08:59:45.069104Z 0 [Note] Shutting down plugin 'sha256_password'
2018-11-29T08:59:45.069107Z 0 [Note] Shutting down plugin 'mysql_native_password'
2018-11-29T08:59:47.223819Z 3 [Note] Access denied for user 'UNKNOWN_MYSQL_USER'@'localhost' (using password: NO)
2018-11-29T09:03:01.302636Z 4 [Note] Access denied for user 'root'@'localhost' (using password: NO)
2018-11-29T09:03:11.638245Z 5 [Note] Access denied for user 'root'@'localhost' (using password: NO)

> mysql -u root -p'密码'
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
> mysql -uusername -p'password'  --default-character-set=utf8 xxxdb
```

# 导出数据

```bash
> mysqldump -uusername -p'password'  --default-character-set=utf8 xxxdb > xxxxdb.sql

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


# mysql 配置优化

```my.cnf
# Default Homebrew MySQL server config
[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect = 'SET NAMES utf8mb4'
# Only allow connections from localhost
bind-address = 127.0.0.1

# mysql写入速度优化
innodb_buffer_pool_size = 1024M
innodb_log_file_size = 512M
innodb_log_buffer_size = 100M
innodb_flush_log_at_trx_commit = 2
innodb_autoextend_increment=512M


# 最大连接数，此项目设备频繁上下线，每秒钟可以达到K台
max_connections=2048
max_connect_errors=100
```

# 最大连接数优化

查看数据库连接数设置是否合理，只需要计算一下 `max_used_connections / max_connections * 100%` ，这个理想值为 `85%`

```bash
# 查看最大连接数
mysql>  show variables like "%connections%";
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| max_connections      | 151   |
| max_user_connections | 0     |
+----------------------+-------+
2 rows in set (0.01 sec)

# 查看已经使用的最大连接数
mysql>  show status like "%connections%";
+-----------------------------------+-------+
| Variable_name                     | Value |
+-----------------------------------+-------+
| Connection_errors_max_connections | 32    |
| Connections                       | 17864 |
| Max_used_connections              | 152   |
+-----------------------------------+-------+
3 rows in set (0.06 sec)

```

设置 `max_connnections`

```bash
# /etc/my.cnf
# 1. 此方式设置并不是立即生效，需要重启mysql
> vi /etc/my.cnf

# 最大连接数
max_connections=2048
max_connect_errors=100

# 2. 使用命令行设置已经运行的 mysql 的最大连接数
mysql> set GLOBAL max_connections=2048;
```


