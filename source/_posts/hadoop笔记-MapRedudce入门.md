---
title: hadoop笔记-MapRedudce入门
date: 2018-12-14 14:17:36
tags:
    - hadoop
    - 大数据
---


# 导入包

可以在 `hadoop` 中找到相应的 `jar` 导入到项目，如果使用 `maven` 需要到 `mvn` 中心去搜索 `hadoop` 关键字，找到相应的包导入到项目。

# 导入配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://centos1/</value>
  </property>
</configuration>
```

```conf
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

# 编写代码

```java
package com.code.hadoop.mr;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.util.StringTokenizer;

/**
 * Created by Cody on 2018/12/2.
 *
 * MapReduce
 *
 */
public class WordCount {
    /**
     * mapper 类
     */
    public static class TokenizerMapper extends Mapper<Object,Text,Text,IntWritable>{
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        @Override
        protected void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            StringTokenizer tokenizer = new StringTokenizer(value.toString());
            while (tokenizer.hasMoreTokens()){
                // 按照空格分割单次
                word.set(tokenizer.nextToken());
                // 单次计数 +1
                context.write(word,one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text,IntWritable,Text,IntWritable>{

        private IntWritable result = new IntWritable(0);

        @Override
        protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int sum = 0;
            for(IntWritable val : values){
                sum += val.get();
            }
            result.set(sum);
            context.write(key,result);
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        System.setProperty("HADOOP_USER_NAME", "root");
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        job.setJobName("WorldCountApp");
        job.setJarByClass(WordCount.class);
        // 设置mapper、reducer类
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        // 设置reducer个数
        // job.setNumReduceTasks(1);
        // 设置输入格式
        job.setInputFormatClass(TextInputFormat.class);
        // 设置输出的key、value类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job,new Path(args[0]));
        FileOutputFormat.setOutputPath(job,new Path(args[1]));

        int status = job.waitForCompletion(true) ? 0:1;
        System.out.println(status);
    }


}

```

# 本地运行

在编辑器里面设置运行时的输入参数 `file:///e:/test.txt`,`file:///e:/out` ，运行项目后，会在本地 `e:/out` 下面有结果的输出文件。需要确保 `text.txt` 存在，并且 `out` 目录不存在。

# 远程调试运行

在编辑器里面设置运行时的输入参数 `hdfs://centos1/data/hello.txt`,`hdfs://centos1/data/result`，需要确保 `hello.txt` 中有一定的数据量，并且 `result` 目录不存在。



