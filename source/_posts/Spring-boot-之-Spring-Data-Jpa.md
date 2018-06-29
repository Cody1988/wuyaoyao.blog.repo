---
title: Spring boot 之 Spring Data Jpa
date: 2018-06-29 17:31:05
tags:
  - spring boot
  - spring data jpa
---

## maven依赖

```pom
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 事务
`spring boot`开启事务需要在入口`Application`类上添加`@EnableTransactionManagement`注解。通常我们在使用多个数据库`插入`,`删除`,`更新`操作的时候，需要使用`事务`来保证数据的正确，在`spring boot`中可以将`@Transactional`注解添加到`service`类或者类的方法上，这样框架在调用的时候，就会通过`AOP`来使`事务`生效。但是这样有一个不友好的地方，那就是，我们需要频繁的在每一个数据库操作的方法上添加`@Transactional`注解。

## 使用@Aspect实现自动添加事务
为了方便事务处理，可以使用`@Aspect`注解，通过`AOP`机制，来为相应的操作，添加合适的事务。具体实现如下:

```java

@Aspect
@Configuration
public class TxAdviceInterceptor {

    private static final String AOP_POINTCUT_EXPRESSION = "execution (* com..*.service.*.*(..))";

    @Autowired
    private PlatformTransactionManager transactionManager;

    @Bean
    public TransactionInterceptor txAdvice() {
        NameMatchTransactionAttributeSource source = new NameMatchTransactionAttributeSource();
        // 只读事务
        RuleBasedTransactionAttribute readOnly = new RuleBasedTransactionAttribute();
        readOnly.setReadOnly(true);
        readOnly.setPropagationBehavior(TransactionDefinition.PROPAGATION_SUPPORTS);
        // 读写事务
        RuleBasedTransactionAttribute require = new RuleBasedTransactionAttribute();
        require.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

        Map<String, TransactionAttribute> attrs = new HashMap<>();
        // 增删改上应用可读写事务，并且异常时rollback
        attrs.put("add*", require);
        attrs.put("delete*", require);
        attrs.put("remove*", require);
        attrs.put("update*", require);
        attrs.put("patch*", require);
        // 非增删改时只读
        attrs.put("*", readOnly);
        source.setNameMap(attrs);
        TransactionInterceptor txAdvice = new TransactionInterceptor(transactionManager, source);
        return txAdvice;
    }

    @Bean
    public Advisor txAdviceAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression(AOP_POINTCUT_EXPRESSION);
        return new DefaultPointcutAdvisor(pointcut, txAdvice());
    }

}

```

通过这样的一个`AOP`类，框架会自动在`service`包以及子包下类的方法上添加`事务`，以`add`,`delete`,`remove`等相关名称开头的，添加`读写事务`，其他方法添加`只读事务`。

## 坑
默认`spring data jpa` 创建`Mysql`数据库表是使用的`MyISAM`引擎，而这个引擎的表是不支持`事务`的，因此需要修改配置，将默认引擎改为`InnoDB`

```yml
spring:
  jpa:
    show-sql: true
    generate-ddl: true
    hibernate:
      ddl-auto: update
    # spring jpa默认创建的表使用myasm引擎的，不支持事务处理，必须采用innodb引擎
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
```

## timestamp默认值
几乎每一个数据库表，都会有一个`create_time`，而这个字段，我们在创建表的时候，我们通常会使用`TIMESTAMP DEFAULT CURRENT_TIMESTAMP`生成默认的时间戳，`spring data jpa`的字段也可以通过注解达到相同的效果，而且会自动在创建表的时候添加字段约束。

```java
@Column(
    updatable = false,
    columnDefinition = "TIMESTAMP DEFAULT CURRENT_TIMESTAMP"
)
@CreationTimestamp
@Temporal(TemporalType.TIMESTAMP)
private Date createTime;
```

## ManyToOne
对于多对一的情况通常不需要创建中间表，只需要在`多`的一方创建一个外键关联即可。

```
@ManyToOne
@JoinColumn(name = "category_id")
private ApkCategory category;
```

## 分页查询
在涉及到分页查询的时候，我通常使用`JpaRepository`，它提供了两个非常友好的`findAll`方法，利用这个方法，可以传递`Example`对象，来查找拥有相同的值得数据。
例如，我们有一个`Apk`对象，代表Android apk的数据模型，它的定义如下：

```java
@Entity
@Table(name = "t_apk")
public class Apk implements Serializable{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false,length = 20)
    private String name;
    @Column(unique = true,nullable = false,length = 128)
    private String packageName;
    @ManyToOne
    @JoinColumn(name = "category_id")
    private ApkCategory category;
    @Column
    private String description;
    // 官网地址
    @Column(length = 512)
    private String homepage;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPackageName() {
        return packageName;
    }

    public void setPackageName(String packageName) {
        this.packageName = packageName;
    }

    public ApkCategory getCategory() {
        return category;
    }

    public void setCategory(ApkCategory category) {
        this.category = category;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getHomepage() {
        return homepage;
    }

    public void setHomepage(String homepage) {
        this.homepage = homepage;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```
那么，我们需要根据条件查找Apk，那么只需要根据条件，创建一个`Apk`对象，然后就可以查出对应条件的所有的`Apk`了，比如我想查找`homepage`为`http://wuyaoyao.xyz`的`Apk`列表，那么就这样设置查询条件:

```java
Apk filter = new Apk();
filter.setHomepage('http://wuyaoyao.xyz');
Pageable pageable = PageRequest.of(0,20, Sort.Direction.ASC,"id");
Page<Apk> result = apkRepository.findAll(Example.of(apk),pageable);
// 获取查询出的列表数据
List<Apk> list = result.getContent();
```

如果需要获取分页数据的`当前页`，`总页数`等内容，可以从`Page`对象中获取。











