---
title: hadoop笔记-安装配置
date: 2018-12-14 14:13:44
tags:
  - hadoop
  - 大数据
---


# 前言

## 准备

1. 下载 `hadoop-2.8.4 `的压缩包，并解压到 `/usr/local/software` 目录；
2. 安装 `ssh` / `rsync`，准备几台服务器或者虚拟机；

## 配置

`hadoop` 的相关配置文件在其目录下的 `./etc/hadoop` 中，要正确运行起集群需要做一些配置


hadoop-env.sh
---
这个文件，需要设置一下 `JAVA_HOME`、`HADOOP_LOG_DIR`

```conf
# java home
export JAVA_HOME=/usr/local/software/jdk1.8
# hadoop log dir , 首先需要保证目录已经存在
export HADOOP_LOG_DIR=/var/log/hadoop
```

yarn-env.sh
---

```conf
YARN_LOG_DIR=/var/log/hadoop
```

core-site.xml
---

```xml
<!-- 指定NameNode URI -->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://server1/</value>
</property>

```

hdfs-site.xml
---

`HDFS NameNode` 配置，首先创建文件目录 `/data/hadoop`

配置文件修改：
```xml
<configuration>
    <!-- NameNode 存储 Namespace 和事务日志的 linux 文件系统的目录 -->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/data/hadoop/namenode</value>
    </property>
    <!-- DataNode 在 linux 文件系统存储 block 的目录 -->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/data/hadoop/datanode</value>
    </property>
    <!-- 文件拷贝数量 -->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
</configuration>
```

mapred-site.xml
---

`MapReduce` 应用配置

```xml
<configuration>
    <!-- Execution framework set to Hadoop YARN. -->
　　<property>
　　　　<name>mapreduce.framework.name</name>
　　　　<value>yarn</value>
　　</property>
</configuration>
```

yarn-site.xml
---

```xml
<configuration>
    <!-- ResourceManager 主机，使用默认的ResourceManager组件的端口 -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<!-- master 的ip地址 -->
		<value>192.168.56.100</value>
	</property>
	<!-- Map/Reduce应用的Shuffle服务 -->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
</configuration>
```

slaves
---
假如有四台主机，确保可以通过 `ssh` 相互登录，此处是创建了四台虚拟机，并安装 `CentOs` 系统，设置各个主机的静态 `IP` 地址。

在 `/etc/hosts` 文件中添加一下配置项：

```conf
192.168.74.138 server1
192.168.74.139 server2
192.168.74.140 server3
192.168.74.141 server4
```

配置 `slaves` 文件

```bash
> vi hadoopxxx/etc/hadoop/slaves

server2
server3
server4

```

xcall 与 xsync配置
---

xcall.sh

```sh
#!/bin/bash

params=$@
i=1
for (( i=1 ; i <= 4 ; i = $i + 1 )) ; do
    echo ============= server$i $params =============
    ssh server$i "$params"
done
```

保存文件，并添加可执行权限，将文件拷贝到  `/usr/local/bin` 目录下； 然后使用命令行为所有的主机安装 `rsync`

```bash
xcall.sh yum install rsync
```

所有主机安装完成后，创建 `xsync.sh`

xsync.sh

```sh
#!/bin/bash

if [[ $# -lt 1 ]] ; then echo no params ; exit ; fi

p=$1
#echo p=$p
dir=`dirname $p`
filename=`basename $p`
#echo filename=$filename
cd $dir
fullpath=`pwd -P .`
#echo fullpath=$fullpath

user=`whoami`
for (( i = 1 ; i <= 4 ; i = $i + 1 )) ; do
   echo ======= server$i =======
   rsync -lr $p ${user}@server$i:$fullpath
done ;
```


创建jps的软连接到 `/usr/local/bin` 下

```bash
ln -s /usr/local/software/jdk/bin/jps /usr/local/bin/jps
```



# 格式化

```bash
hadoop namenode -format
```

# 启动

```bash
start-all.sh
```

# 查看所有的主机执行情况

```bash
xcall.sh jps
```

# 网页界面


服务 | web界面| 说明
---|---|---
NameNode | http://nn_host:port/ | 默认端口50070
ResourceManager | http://rm_host:port/ | 默认端口8088
MapReduce JobHistory Server	| http://jhs_host:port/ | 默认端口19888

