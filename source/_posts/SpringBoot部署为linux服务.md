---
layout: spring
title: spring boot部署为linux服务
date: 2017-12-14 17:01:47
tags:
  - spring boot
---

## 添加pom.xml配置项
```
<build>                                                          
	<plugins>                                                    
		<plugin>                                                 
			<groupId>org.springframework.boot</groupId>          
			<artifactId>spring-boot-maven-plugin</artifactId>    
			<configuration>                                      
				<executable>true</executable>                    
			</configuration>                                     
		</plugin>                                                
	</plugins>                                                   
</build>                                                     
```

## 项目打包
使用ide或者mvn命令打包应用
```
mvn package
```

## 创建linux服务

```bash
ln -s /var/springbootapp/myapp.jar /etc/init.d/myapp
```
这样将jar创建为系统的服务了，查看/etc/init.d/myapp 可以看到，这是一个shell脚本
查看myapp脚本是否有执行的权限，如果没有，则使用chmod命令修改

## 按照系统服务运行项目

```bash
service myapp start | stop | restart
```

## 更新项目
下次更新项目只需要把jar包替换掉，然后执行restart就可以了