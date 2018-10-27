---
title: mysql-emoji表情
date: 2018-10-27 18:35:44
tags:
  - mysql
  - 微信昵称保存
---

# mysql emoji表情

配置 my.cnf

```cnf
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

```
配置完成后重启`mysqld`,再次连接并查看编码

```mysql bash
SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%';
```
如果还没有成功可以执行命令
```mysql bash
set names utf8mb4;
```