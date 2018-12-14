---
title: mongodb笔记-安装与配置
date: 2018-12-14 14:23:33
tags:
    - mongodb
    - 数据库
---


# 安装

``` bash
# 解压
> tar xzvf mongodb-linux-x86_64-v4.0-latest.tgz
# 拷贝到/usr/local下
> mv mongodb-linux-x86_64-4.0.1-80-g44b5c11973 /usr/local/mongodb4.0.1
# 配置环境变量
> vi ~/.bash_profile

# mongodb
export MONGODB_HOME=/usr/local/mongodb4.0.1
export PATH=$PATH:$MONGODB_HOME/bin

# 让环境配置生效
> source ~/.bash_profile

# 查看mongo版本
> mongo --version

MongoDB shell version v4.0.1-80-g44b5c11973
git version: 44b5c1197367d6cda1ad75d2f0860c3d3683b86e
allocator: tcmalloc
modules: none
build environment:
    distarch: x86_64
    target_arch: x86_64



```
# mongodb 配置

创建存储位置、log目录并启动 `mongod` 服务

```bash

# 创建存储位置
> mkdir -p /data/mongodb/data/db
# 创建log目录
> mkdir -p /var/log/mongodb

# 启动 mongod，加上 --fork 参数表示后台启动
> mongod --dbpath /data/mongodb/data/db --logpath=/var/log/mongodb/mongod.log --logappend
# 后台启动
> mongod --dbpath /data/mongodb/data/db --logpath=/var/log/mongodb/mongod.log --logappend --fork

# 连接 mongod 服务
> mongo

```

## 创建root用户

经过以上步骤，就可以对 `mongodb` 进行操作了，在实际项目中，通常需要创建 `root`用户，以及各个库的管理员账号。

```bash

mongo> use admin
switched to db admin

# 创建root
mongo> db.createUser({user: "root",pwd:"123456",roles:["root","userAdminAnyDatabase"]})
Successfully added user: { "user" : "root", "roles" : [ "root", "userAdminAnyDatabase" ] }

# 查看创建的用户
mongo> db.system.users.find().pretty();
{
	"_id" : "admin.root",
	"user" : "root",
	"db" : "admin",
	"credentials" : {
		"SCRAM-SHA-1" : {
			"iterationCount" : 10000,
			"salt" : "H8uuWFHR/Gc/g1m+68HfXw==",
			"storedKey" : "kPhAKKeOBXKyRLQEF1RNMU61x5A=",
			"serverKey" : "+NWpzT1U710X/QsmeDwmfvy84mo="
		},
		"SCRAM-SHA-256" : {
			"iterationCount" : 15000,
			"salt" : "WY6weLUQ65phyhnLXhc5BKb+IXRMzpR8kK98Ew==",
			"storedKey" : "VNfejvGnhNPQVB3BmRtBkB+uw761Qa9trVdg+iyaXfs=",
			"serverKey" : "DVkHNrhSDspAUg76V3vioJaFhlHpyLTlVi7iO2BJuYQ="
		}
	},
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		},
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}


```

## 使用配置文件设置 mongod 启动参数

首先关闭之前开启的 `mongod`

```bash
# 关闭 mongod
mongo> db.shutdownServer();
```

### 创建配置文件并编辑

```bash
> vi /etc/mongodb.conf
```

配置文件内容，[配置文件原文位置](http://docs.mongodb.org/manual/reference/configuration-options/)

```conf
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# 日志文件存储位置
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# 数据存储位置
storage:
  dbPath: /data/mongodb/data/db
# journal:
#   enabled: true
#  engine:
#  wiredTiger:

# 后台启动
processManagement:
  fork: true  # fork and run in background
# pidFilePath: /var/run/mongodb/mongod.pid
# timeZoneInfo: /usr/share/zoneinfo

# 绑定到所有的网卡
net:
  port: 27017
  bindIp: 0.0.0.0  


#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options

#auditLog:

#snmp:


```

### 使用配置文件启动 mongod

编辑好配置文件之后，就可以使用配置文件来设置 `mongod` 启动时的参数了。

```bash
# 启动 mongod
> mongod --config /etc/mongod.conf --auth
about to fork child process, waiting until server is ready for connections.
forked process: 4679
child process started successfully, parent exiting

```

# 项目中使用

1. 创建数据库
```bash
> use hello
```

2. 创建用户

```bash
mnogo> use admin;
# 授权
mongo> db.auth("root","123456");
# 创建用户
mongo> db.createUser(
  {
    user: "hello",
    pwd: "hello123",
    roles: [{ role: "readWrite", db: "hello_db" },"dbAdmin","dbOwner"]
  }
)

Successfully added user: {
	"user" : "hello",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "hello"
		},
		"dbAdmin",
		"dbOwner"
	]
# 创建的用户登录
mongo> exit
> mongo
mongo> use hello_db
# 登录授权
mongo> db.auth("hello","hello123")
```

这样就创建好了一个数据库 `hello_db`，并创建了这个数据库有读写权限的管理员 `hello`

# mongodb 系统服务配置

创建配置文件 `/etc/rc.d/init.d/mongod`，请添加脚本

```script

start() {
   /usr/local/mongodb4.0.1/bin/mongod  --config /etc/mongod.conf --auth
}

stop() {
    /usr/local/mongodb4.0.1/bin/mongod  --config /etc/mongod.conf --shutdown
}

case "$1" in
  start)
 start
 ;;

stop)
 stop
 ;;

restart)
 stop
 start
 ;;
  *)
 echo
$"Usage: $0 {start|stop|restart}"
 exit 1
esac


```

保存脚本，并添加可执行权限

```bash
> chmod +x /etc/rc.d/init.d/mongod
```

使用 `service` 执行服务

```bash
# 启动
> service mongod start

# 停止
> service mongod stop

# 重启
> service mongod restart

```



[官方文档-创建用户](https://docs.mongodb.com/manual/reference/method/db.updateUser/)

[官方文档-MongoDB支持的角色](https://docs.mongodb.com/manual/reference/built-in-roles/index.html#database-user-roles)