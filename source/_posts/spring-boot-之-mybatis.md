---
title: spring boot 之 mybatis
date: 2018-07-11 18:52:38
tags:
  - spring boot
  - mybatis
---

# 前言

目前在使用`spring boot`进行后台开发中，`orm`库使用的比较多的是`spring data jpa`，`hibernate`以及`mybatis`。我使用的最多的是`mybatis`，对应其它两种虽然很简洁但是在使用中老是陷入到各种纠结之中，最终还是喜欢`mybatis`这种操作感。

# 集成

`spring boot`集成`mybatis`

## 版本属性
```pom
<properties>
    <mybatis.stater.version>1.3.1</mybatis.stater.version>
    <pagehelper.version>1.2.3</pagehelper.version>
</properties>
```
## 库引入

```xml pom
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>${mybatis.stater.version}</version>
</dependency>
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>${pagehelper.version}</version>
</dependency>
```

## spring boot application注解配置

```java
@SpringBootApplication
@EnableTransactionManagement
@MapperScan("com.cody.mickey.dao")
public class MickeyServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(MickeyServerApplication.class, args);
	}
}
```

## spring boot 配置文件配置
`spring boot` 配置文件配置，其中指定了使用的数据库`用户名`，`密码`，`数据库名称`，`mapper`文件位置，`mybatis config`文件位置
```yml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: db_username
    password: db_password
    url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=UTF-8&useSSL=false
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml
  config-location: classpath:mybatis/mybatis-config.xml
  type-aliases-package: com.cody.mickey.entity;com.cody.mickey.form
```

## mybatis-config.xml
所有的 `mapper` 文件，以及 `mybatis-config.xml`文件都放在 `resources/mybatis`下。 `mybatis-config.xml` 中定义了一些常用的类型，这样就不需要再`mapper`文件中写这些类的全称了

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 将下划线转为驼峰写法 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="logImpl" value="STDOUT_LOGGING" />
    </settings>
    <typeAliases>
        <typeAlias alias="Integer" type="java.lang.Integer" />
        <typeAlias alias="Long" type="java.lang.Long" />
        <typeAlias alias="HashMap" type="java.util.HashMap" />
        <typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap" />
        <typeAlias alias="ArrayList" type="java.util.ArrayList" />
        <typeAlias alias="LinkedList" type="java.util.LinkedList" />
    </typeAliases>
</configuration>
```

以上就完成了基本的配置

# 我的实践

在实际项目中，通常对数据的处理包括`增删改查`4种，所以我在项目中，通常会定义一个`Dao`接口，其内容如下：

```java
public interface IDao<T> {

    T findOne(Object filter);

    List<T> list(Object filter);

    void add(Object bean);

    void remove(Object filter);

    void update(Object bean);
}

```
泛型为集成此接口的 `Dao` 接口所针对的 `bean` 类。假如有一个Android的应用管理的接口需要做，那么我们定义一个 `Apk` bean 类，然后再定义一个对数据库进行操作的接口 `ApkDao`。

## Apk.java

```java
public class Apk implements Serializable{
    private Long id;
    private String name;
    private String packageName;
    private ApkCategory category;
    private String description;
    // 官网地址
    private String homepage;
    private String secretId;
    private User user;

    private List<Scenario> scenarios;

    // getters and setters
}

```

## ApkDao.java

`dao` 接口只需要继续 `IDao`即可，这样就默认有 `IDao` 中定义的5个方法，然后我们只需要对我们需要用到的方法，在 `AppMapper.xml` 中定义接口对应的 `sql` 操作即可

```java
public interface ApkDao extends IDao<Apk> {
}
```
## ApkMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.cody.mickey.dao.ApkDao" >

    <resultMap id="bean" type="Apk" autoMapping="true">
    </resultMap>

    <sql id="selectSql">
        SELECT * FROM t_apk
    </sql>

    <insert id="add" keyProperty="id" useGeneratedKeys="true">
        INSERT INTO t_apk(
            description
            ,homepage
            ,name
            ,package_name
            ,category_id
            ,user_id
            ,secret_id
        )
        VALUES(
            #{description}
            ,#{homepage}
            ,#{name}
            ,#{packageName}
            ,#{category.id}
            ,#{user.id}
            ,#{secretId}
        )
    </insert>
    <update id="update">
        update t_apk
        <set>
            <if test="description != null">description=#{description}</if>
            <if test="homepage != null">homepage=#{homepage}</if>
            <if test="package_name != null">package_name=#{packageName}</if>
        </set>
        WHERE id=#{id}
    </update>

    <delete id="remove">
        DELETE FROM t_apk WHERE id=#{id}
    </delete>

    <select id="list" resultMap="bean">
        SELECT * FROM t_apk WHERE user_id=#{user.id}
    </select>

    <select id="findOne" resultMap="bean">
        <include refid="selectSql"></include>
        <where>
            <choose>
                <when test="id != null">id=#{id}</when>
                <when test="secretId != null">secret_id=#{secretId}</when>
                <otherwise>1=-1</otherwise>
            </choose>
        </where>
        LIMIT 1
    </select>
</mapper>
```
以上就定义好了对 `Apk` 进行增删改查的操作，如果还需要其他的操作，可以自行在 `ApkDao` 中添加方法，并在对应的 `mapper` 中写 `sql` 操作语句。

