---
title: spring boot 之 actuator监控应用状态
date: 2018-07-02 17:39:25
tags:
  - spring boot
  - actuator
---

# 监测管理

`Actuator`是`Spring Boot`提供的对应用系统的自省和监控的集成功能，可以对应用系统进行配置查看、相关功能统计等。


# maven依赖

```pom
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

# 说明
spring boot 2.0之后，只默认开启两个配置，如果需要开启更多需要在配置文件中配置，开启所有的功能

```yml
management:
  endpoints:
    web:
      exposure:
        include: '*'
    enabled-by-default: true
```

# 查看支持的服务

打开地址`http://${ip}:${port}/actuator`可以查看支持的服务类型。


```json
{
    "_links": {
        "self": {
            "href": "http://localhost:20005/actuator",
            "templated": false
        },
        "auditevents": {
            "href": "http://localhost:20005/actuator/auditevents",
            "templated": false
        },
        "beans": {
            "href": "http://localhost:20005/actuator/beans",
            "templated": false
        },
        "health": {
            "href": "http://localhost:20005/actuator/health",
            "templated": false
        },
        "conditions": {
            "href": "http://localhost:20005/actuator/conditions",
            "templated": false
        },
        "shutdown": {
            "href": "http://localhost:20005/actuator/shutdown",
            "templated": false
        },
        "configprops": {
            "href": "http://localhost:20005/actuator/configprops",
            "templated": false
        },
        "env": {
            "href": "http://localhost:20005/actuator/env",
            "templated": false
        },
        "env-toMatch": {
            "href": "http://localhost:20005/actuator/env/{toMatch}",
            "templated": true
        },
        "info": {
            "href": "http://localhost:20005/actuator/info",
            "templated": false
        },
        "logfile": {
            "href": "http://localhost:20005/actuator/logfile",
            "templated": false
        },
        "loggers": {
            "href": "http://localhost:20005/actuator/loggers",
            "templated": false
        },
        "loggers-name": {
            "href": "http://localhost:20005/actuator/loggers/{name}",
            "templated": true
        },
        "heapdump": {
            "href": "http://localhost:20005/actuator/heapdump",
            "templated": false
        },
        "threaddump": {
            "href": "http://localhost:20005/actuator/threaddump",
            "templated": false
        },
        "metrics-requiredMetricName": {
            "href": "http://localhost:20005/actuator/metrics/{requiredMetricName}",
            "templated": true
        },
        "metrics": {
            "href": "http://localhost:20005/actuator/metrics",
            "templated": false
        },
        "scheduledtasks": {
            "href": "http://localhost:20005/actuator/scheduledtasks",
            "templated": false
        },
        "httptrace": {
            "href": "http://localhost:20005/actuator/httptrace",
            "templated": false
        },
        "mappings": {
            "href": "http://localhost:20005/actuator/mappings",
            "templated": false
        }
    }
}
```

打开相应的地址，就可以查看信息，这里需要注意，如果项目中使用了权限控制，为了安全性，可以给连接设置权限，只有系统管理员可以查询相应的信息；

如果使用了`nginx`进行反向代理了，那么`upstream`名称，应该就是域名，否则可能返回不正确的url连接。


[TO: 不错的actuator使用说明](http://www.baeldung.com/spring-boot-actuators)







