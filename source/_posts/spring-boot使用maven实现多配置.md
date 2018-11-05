---
title: spring boot 使用maven profiles实现多配置
date: 2018-11-05 20:31:46
tags:
  -- spring boot
  -- maven 
---
# 说明

通常在项目开发中，我们需要在不同的运行环境中运行，如果:`dev`，`test`，`snapshot`，`release`等。`Spring Boot`项目中在不同的环境下面，可能同一个配置参数会有不同的值，我们通常将这些值配置到不同环境下对应的文件中。比如通常情况下，通用的配置都放在`application.properties`，而不同环境下对应的配置为`application-xxx.properties`；比如`dev`环境下对应的配置为`application-dev.properties`。


# 如何是配置文件生效

在不做任何处理的情况下启动 `spring boot`应用，只有配置 `application.properties`    文件被加载，如果希望加载不同的配置文件那么需要配置`spring.profiles.active=xxx`，其中`xxx`对应`dev`,`test`等。

通常可以在运行的命令行中制定参数，来使配置生效，比如:

```bash
java -jar --spring.profiles.active=dev
```

具体可以参考[spring boot 之多配置文件](http://blog.wuyaoyao.xyz/2018/07/13/spring-boot-%E4%B9%8B%E5%A4%9A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6/#more)

# 使用`maven`配置文件`pom.xml`类配置，并执行不同的配置

首先在`pom.xml`中添加`profiles`配置项，它是与`<dependencies>`是平级的。

```pom.xml
<profiles>
	<profile>
		<id>dev</id>
		<properties>
			<filter.env>dev</filter.env>
		</properties>
		<activation>
			<activeByDefault>true</activeByDefault>
		</activation>
	</profile>
	<profile>
		<id>snapshot</id>
		<properties>
			<filter.env>dev</filter.env>
		</properties>
	</profile>
	<profile>
		<id>prod</id>
		<properties>
			<filter.env>prod</filter.env>
		</properties>
	</profile>
</profiles>
```
在`<build>`下添加配置

```pom.xml
<resources>
	<resource>
		<directory>src/main/resources</directory>
		<filtering>true</filtering>
	</resource>
	<resource>
		<filtering>true</filtering>
		<directory>src/main/resources</directory>
		<includes>
			<include>application-${filter.env}.properties</include>
			<include>application.properties</include>
		</includes>
	</resource>
</resources>

```

在`application.properties`中添加配置

```conf
spring.profiles.active=@filter.env@
```
其中 `@filter.env@`为指定的环境。


这个配置是覆盖默认的`resources`加载方式，此处会加载`application-properties`以及`application-xxx.properties`配置文件。
其中参数`filter.env`为上面`profile`中定义的，我们再运行的时候，会指定。此时，`maven`配置中就包括了多个配置，如果需要指定某个配置进行打包，则可以使用命令行`mvn clean package -P xxx`比如 `dev`环境为

```bash
mvn package -P dev
```

即可以使用指定的配置进行打包。


# 在intellij中使用特定配置

`intellij` 对`maven`的多`profile`有很好的支持。如果要运行哪个环境的配置，只需要在`Maven Projects` -> `Profiles`中勾选响应的配置即可。

<img width="500" src="https://ws1.sinaimg.cn/large/627047c4gy1fwxgkadj8ej20nm0mkq4m.jpg" />
