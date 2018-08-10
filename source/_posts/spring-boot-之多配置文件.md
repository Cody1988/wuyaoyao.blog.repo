---
title: spring boot 之多配置文件
date: 2018-07-13 17:41:32
tags:
  - spring boot
---

# 前言

在后台开发中，我们经常遇到正式发布的时候和开发环境以及测试环境下面，配置不一样的情况。比如：数据库地址、用户名、密码等等。在 `spring boot` 的开发中，可以使用多配置文件来解决。

`spring boot` 默认加载的是 `application.property` 或者 `application.yml`，我习惯使用 `application.yml` 这种方式更符合我的阅读以及编码习惯。那么在开发环境下可以增加一个 `application-dev.yml` ,在此文件中写入开发环境下生效的配置信息。

# 运行

在开发环境下我们通常使用开发工具运行项目，我使用的是 `intellij idea` 工具，只需要进行简单的配置就可以使配置生效。

`Run/Debug Configurations -> Spring Boot -> Configuration -> Program arguments: `

在这个配置项下面添加 `--spring.profiles.active=dev` 即可，如下图所示。

<p align="center"><img src="/images/common/spring_boot_config_args.jpg" /></p>

在发布的环境下，只需要在运行时添加一下参数即可

```bash
java -jar --spring.profiles.active=dev xxx.jar
```


