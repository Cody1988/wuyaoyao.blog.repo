---
title: 使用docker部署spring boot
date: 2019-06-25 20:18:24
tags:
---

# 使用docker部署spring boot 

## maven配置

设置`.m2/settings.xml`

```settings.xml
<pluginGroups>
    <pluginGroup>com.spotify</pluginGroup>
</pluginGroups>
```

## 基础使用

在项目根目录创建Dockerfile

```Dockerfile
FROM java:latest
# host /var/lib/docker 配置tomcat的目录
VOLUME /tmp
COPY target/*.jar /usr/local/app.jar
ENTRYPOINT ["java","-jar","/usr/local/app.jar"]
```

在项目`pom.xml`的`build->plugins->plugin`添加配置

```pom
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.8</version>
    <configuration>
        <repository>${project.groupId}/${project.artifactId}</repository>
        <tag>${project.version}</tag>
    </configuration>
</plugin>
```
运行`build`

```bash
mvn package dockerfile:build
```

## 深入配置

```Dockerfile
FROM java:latest
# host /var/lib/docker 配置tomcat的目录
VOLUME /tmp
# pom中配置的参数
ARG JAR_FILE
COPY target/${JAR_FILE} /usr/local/app.jar
ENTRYPOINT ["java","-jar","/usr/local/app.jar"]
```

```pom
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.8</version>
    <!--可以运行的命令-->
    <executions>
        <execution>
            <id>default</id>
            <goals>
                <goal>build</goal>
                <goal>push</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <repository>${project.groupId}/${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <!-- docker ARGS -->
        <buildArgs>
            <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

运行`build`

```bash
# 打包
mvn package
# docker image 推送到docker hub
mvn deploy
```
