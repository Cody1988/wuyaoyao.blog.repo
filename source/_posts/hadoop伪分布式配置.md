---
title: hadoop伪分布式配置
date: 2018-02-22 11:04:10
tags:
  - hadoop
  - 大数据
---

# hadoop伪分布式
## 下载
下载Hadoop最新版本，并放到特定的目录，在当前用户的根目录下面配置`.bash_profile`文件，并添加

```bash profile
# hadoop path
HADOOP_HOME=/Users/cody/software/hadoop-3.0.0
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=${HADOOP_HOME}/lib/native"
# end hadoop path
```
## 配置
### etc/hadoop/cor-site.xml
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
	 <property>
	    <name>hadoop.tmp.dir</name>
	    <value>/your/local/path</value>
	    <description>hadoop temp dir</description>
	</property>
</configuration>
```

### etc/hadoop/hdfs-site.xml
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

### ssh免密码登陆

#### 验证
```bash
ssh localhost
```
如果不成功，则需要配置一下，配置方法为将ssh的pub key添加到~/.ssh/authorized_keys文件

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa  
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  
chmod 0600 ~/.ssh/authorized_keys  
```

## 执行

以下展示的是在本地跑一个**MapReduce**任务
### 1. 格式化文件系统
```bash
bin/hdfs namenode -format
```

### 2. 静默启动NameNode和DataNode
```bash
sbin/start-dfs.sh
```
hadoop静默log输出到**$HADOOP_HOME/logs**目录下面

### 3. NameNode网页界面
```url
http://localhost:9870/
```

### 4. 创建执行**MapReduce**任务所需要的HDFS目录
```bash
bin/hdfs dfs -mkdir /user
bin/hdfs dfs -mkdir /user/cody
```

### 5. 将文件拷贝到分布式文件系统
```bash
bin/hdfs dfs -mkdir input
bin/hdfs dfs -put etc/hadoop/*.xml input
```

### 6. 运行demo程序
```bash
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0.jar grep input output 'dfs[a-z.]+'
```
### 7. 检测输出文件
将输出文件从分布式文件系统考到本地系统

```bash
bin/hdfs dfs -get output output
``` 
在分布式系统中查看

```bash
bin/hdfs dfs -cat output/*
```

### 8. 停止运行

```bash
bin/stop-dfs.sh
```

# YARN

## 配置参数
### 1. etc/hadoop/mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
### 2. etc/hadoop/yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

### 3. start-yarn.sh 启动yarn
### 4. stop-yarn.sh 关闭yarn


# 错误修复
# 1. 运行命令行报**warn**错误
```log
WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

解决方案

[连接地址](http://www.cnblogs.com/likui360/p/6558749.html)

# 2. 每次重启机器，都会需要重新执行 hdfs namenode -format 否则无法正常执行


需要在core-site.xml中添加配置项

```xml
<property>
    <name>hadoop.tmp.dir</name>
    <value>/your/local/path</value>
    <description>hadoop temp dir</description>
</property>
```
[解决方案参考](http://blog.csdn.net/zhangt85/article/details/42076207)

