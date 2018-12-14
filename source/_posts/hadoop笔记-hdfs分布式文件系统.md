---
title: hadoop笔记-hdfs分布式文件系统
date: 2018-12-14 14:16:08
tags:
    - hadoop
    - 大数据
---


# hdfs

操作 `hadoop` 分布式文件系统

```bash
# 创建文件夹
> hdfs dfs -mkdir /test

# 上传文件
> hdfs dfs -put hello.txt /test/

# 查看文件列表
>  hdfs dfs -ls -R /
drwxr-xr-x   - root supergroup          0 2018-11-30 09:56 /test
-rw-r--r--   2 root supergroup        129 2018-11-30 09:56 /test/hello.txt

# 查看文件内容
>  hdfs dfs -cat /test/hello.txt
rsync
天气怎么样
hi , how are you
what's the matter
time goes on , never come back
I't the time to go
china is a good place

# 下载文件
> hdfs dfs -get /test/hello.txt ./hello_h.txt

# 删除文件
> hdfs dfs -rm /test/hello.txt

# 删除目录
> hdfs dfs -rm -r /test

```

# java 操作

`java` 操作需要导入各种 `hadoop` 包，此处通过 `maven` 一次性导入所有需要的文件

mvn parent pom文件
---

```pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.cody.hadoop</groupId>
    <artifactId>hadoop-starter</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>hdfs-starter</module>
    </modules>

    <properties>
        <hadoop.version>2.8.4</hadoop.version>
    </properties>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-yarn-common -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-yarn-common</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-mapreduce-client-core -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs-httpfs -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs-httpfs</artifactId>
        </dependency>

    </dependencies>
    <dependencyManagement>
       <dependencies>
           <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs -->
           <dependency>
               <groupId>org.apache.hadoop</groupId>
               <artifactId>hadoop-hdfs</artifactId>
               <version>${hadoop.version}</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
           <dependency>
               <groupId>org.apache.hadoop</groupId>
               <artifactId>hadoop-common</artifactId>
               <version>${hadoop.version}</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-yarn-common -->
           <dependency>
               <groupId>org.apache.hadoop</groupId>
               <artifactId>hadoop-yarn-common</artifactId>
               <version>${hadoop.version}</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-mapreduce-client-core -->
           <dependency>
               <groupId>org.apache.hadoop</groupId>
               <artifactId>hadoop-mapreduce-client-core</artifactId>
               <version>${hadoop.version}</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs-httpfs -->
           <dependency>
               <groupId>org.apache.hadoop</groupId>
               <artifactId>hadoop-hdfs-httpfs</artifactId>
               <version>${hadoop.version}</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/junit/junit -->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>

       </dependencies>
    </dependencyManagement>
</project>

```

 maven 子项目 pom.xml
 
 ```pom.xml
 
 <?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>hadoop-starter</artifactId>
        <groupId>com.cody.hadoop</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>hdfs-starter</artifactId>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>compile</scope>
        </dependency>
    </dependencies>


</project>
 ```
 
 java 测试代码
 ---
 
 
 ```java
 
 package com.cody.hadoop.hdfs;

import org.apache.commons.io.IOUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * 通过hdfs协议，进行hdfs操作；
 * 需要注意HADOOP_USER_NAME需要设置为有访问权限的用户，
 * 否则报权限不足错误
 */
public class HdfsClient {

    private FileSystem fs = null;

    /**
     * 初始化
     * @throws IOException
     */
    @Before
    public void getFs() throws IOException {
        // 设置用户，远程访问的时候，必须设置有访问权限的用户名，否则报出权限不足错误
        System.setProperty("HADOOP_USER_NAME", "root");
        Configuration config = new Configuration();
        config.set("fs.defaultFS","hdfs://centos1:8020/");

        fs = FileSystem.get(config);
    }

    @After
    public void destory() throws IOException {
        fs.close();
    }

    /**
     * 从 hdfs 上传文件
     * @throws IOException
     */
    @Test
    public void upload() throws IOException {
        Path destFile = new Path("hdfs://centos1:8020/test.pdf");
        FSDataOutputStream output = fs.create(destFile);
//        fs.copyFromLocalFile();
        FileInputStream input = new FileInputStream("E:\\大数据\\hadoop02\\Hadoop技术内幕：深入解析YARN架构设计与实现原理.pdf");
        IOUtils.copy(input,output);
    }

    /**
     * 从 hdfs 下载文件到本地
     * @throws IOException
     */
    @Test
    public void download() throws IOException {
        FSDataInputStream inputStream = fs.open(new Path("hdfs://centos1:8020/test.pdf"));
        FileOutputStream outputStream = new FileOutputStream("test.pdf");
        IOUtils.copy(inputStream,outputStream);
//        fs.copyToLocalFile();
    }

    /**
     * 删除当前用户目录下的 a
     * @throws IOException
     */
    @Test
    public void rm() throws IOException {
        fs.delete(new Path("a"),true);
    }

    /**
     * 在当前用户下创建目录
     * @throws IOException
     */
    @Test
    public void mkDirs() throws IOException {
        fs.mkdirs(new Path("a/b/c"));
    }


}

 ```
