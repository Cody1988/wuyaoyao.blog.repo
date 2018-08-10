---
title: mongodb用户管理
date: 2018-08-10 14:27:54
tags:
  - mongodb
---

# 启动mongodb

```bash
mongod
```

# 启动客户端

```bash
mongo --host 127.0.0.1:27017
```

# 设置权限

创建用户，这个用户有权限管理`用户`和`角色`，并且仅可以操作`admin`数据库。

```bash
use admin
db.createUser(
	{
		user:"admin",
		pwd:"abc.123",
		roles:[{role: "userAdminAnyDatabase", db:"admin"}]
	}
)
```

创建一个测试账户，可以读写 `test` 数据库，读 `reporting` 数据库。

```bash
db.createUser(
  {
    user: "test",
    pwd: "abc.123",
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)
```

# 以需要授权的方式启动mongodb

默认启动`mongod`，客户端命令行是可以直接连接操作的，为了安全性，启动`mongod`的时候需要加上`--auth`参数。

```bash
mongod --auth --port 27017 --dbpath /data/db
```

这样，`mongo shell`登入需要用户名和密码，可以使用两种方式登录：连接是授权、连接后授权

## 连接时授权

所谓的`连接时授权`，就是在使用`mongo`命令连接 `mongod` 服务的时候，直接加上用户名和密码。

```bash
mongo -u "admin" -p "abc.123" --authenticationDatabase "admin"
```

## 连接后授权

所有`连接后授权`，就是使用`mongo`命令，连接上`mongod`服务后，在使用用户名密码登录

```bash
mongo
> use test
> db.auth("test","abc.123")
```

[官方文档传送门](https://docs.mongodb.com/manual/tutorial/enable-authentication/)