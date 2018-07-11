---
title: spring boot 之使用shiro对restful接口进行认证管理
date: 2018-06-29 17:38:51
tags:
---
# 前言

目前的开发中，会将`后台`和`前端`严格的分离开，前端通过通过`restful`接口，向后台发送请求或者数据。我们之前在使用`shiro`的使用，当用户未登录的时候访问网站，通常会自动跳转到登录页面，访问出错的时候会跳转到错误页面。但是这样的交互设计，在`restful`的数据交换设计中受到了挑战。在`restful`中，返回的数据应该总是`json`格式的，因此我们需要对`shiro`进行一些配置。

# maven 依赖

```pom 
<!-- shiro 鉴权以及权限控制 -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>${shiro.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>${shiro.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>${shiro.version}</version>
</dependency>
```

# 登录Controller

```java

/**
 * 登录controller
 */
@RestController
@RequestMapping("access")
public class AccessController {
    @RequestMapping(value = "", method = RequestMethod.GET)
    public Object get(@Autowired ApiMessage apiMessage) {
        User user = (User) SecurityUtils.getSubject().getPrincipal();
        apiMessage.setResult(user);

        return apiMessage;
    }
    @PostMapping("logout")
    public Object logout(@Autowired ApiMessage apiMessage){
        Subject subject = SecurityUtils.getSubject();
        if (subject != null) {
            subject.logout();
        }
        return apiMessage;
    }

    @PostMapping("login")
    public Object login(@RequestBody UserForm user, @Autowired ApiMessage apiMessage){
        Subject subject = SecurityUtils.getSubject();
        String username = user.getUsername();
        String password = user.getPassword();
        UsernamePasswordToken token = new UsernamePasswordToken(username, MD5Encode.encode(password));
        token.setRememberMe(true);
        try {
            // 调用shiro登录
            subject.login(token);
            if (subject.isAuthenticated()) {
                apiMessage.setMessage("登录成功");
                // 设置session、设置cookie
                User result = (User) SecurityUtils.getSubject().getPrincipal();
                result.setPassword(null);
                apiMessage.setResult(result);
            } else {
                apiMessage.setSuccess(false);
                apiMessage.setMessage("登录失败");
            }
        } catch (Exception e) {
            apiMessage.setSuccess(false);
            apiMessage.setMessage("用户名或者密码失败");
        }
        return apiMessage;
    }

}
```

# shiro登录认证

```java

/**
 * Created by Cody on 2017/10/23.
 */
public class MyShiroRealm extends AuthorizingRealm {

    @Autowired
    @Lazy
    private UserService userService;

    /**
     * 授权认证、即用户是否有相关的权限
     *
     * @param principals
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

        // 角色、权限
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
//        User user = (User) principals.getPrimaryPrincipal();
//        for (Role role : user.getRoles()) {
//            info.addRole(role.getName());
//            for (Permission p : role.getPermissions()) {
//                info.addStringPermission(p.getName());
//            }
//        }

        return info;
    }

    /**
     * 登录认证
     *
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {

        String username = (String) authenticationToken.getPrincipal();

        User filter = new User();
        filter.setUsername(username);

        User user = userService.findOne(filter);
        if (user == null) return null;

        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(
                user,
                user.getPassword(),
                getName()
        );
        return authenticationInfo;
    }
}

```

# config

```java

/**
 * Created by Cody on 2017/10/23.
 */
@Configuration
public class ShiroConfig {


    /**
     * reference : https://stackoverflow.com/questions/25241801/how-to-configure-shiro-with-spring-boot
     * Map<String, Filter> filters = new HashMap<String, Filter>();
     * filters.put("anon", new AnonymousFilter());
     * filters.put("authc", new FormAuthenticationFilter());
     * filters.put("logout", new LogoutFilter());
     * filters.put("roles", new RolesAuthorizationFilter());
     * filters.put("user", new UserFilter());
     *
     * @param securityManager
     * @return
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 过滤器
        ShiroAuthFilter shiroAuthFilter = new ShiroAuthFilter();
        Map<String, Filter> filters = new LinkedHashMap<>();
        filters.put("authc", shiroAuthFilter);
        shiroFilterFactoryBean.setFilters(filters);
        // 拦截器
        Map<String, String> filter = new LinkedHashMap<>();
        filter.put("/static/**", "anon");
        //登录退出不需要权限
        filter.put("/access/login", "anon");
        filter.put("/swagger*/**", "anon");
        filter.put("/webjars/**", "anon");
        filter.put("/v2/**", "anon");

        filter.put("/**", "authc");
//        shiroFilterFactoryBean.setLoginUrl("/access/needLogin");
//        shiroFilterFactoryBean.setUnauthorizedUrl("/403");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filter);

        return shiroFilterFactoryBean;
    }

    /**
     * 凭证匹配器
     * （由于我们的密码校验交给Shiro的SimpleAuthenticationInfo进行处理了
     * ）
     *
     * @return
     */
    @Bean
    public MyShiroRealm myShiroRealm() {
        MyShiroRealm myShiroRealm = new MyShiroRealm();
//        myShiroRealm.setCredentialsMatcher(hashedCredentialsMatcher());

        return myShiroRealm;
    }

    @Bean
    public DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(myShiroRealm());

        return securityManager;
    }

    /**
     * 开启shiro aop注解支持.
     * 使用代理方式;所以需要开启代码支持;
     *
     * @param securityManager
     * @return
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }

    @Bean(name = "simpleMappingExceptionResolver")
    public SimpleMappingExceptionResolver
    createSimpleMappingExceptionResolver() {
        SimpleMappingExceptionResolver r = new SimpleMappingExceptionResolver();
        Properties mappings = new Properties();
        mappings.setProperty("DatabaseException", "databaseError");//数据库异常处理
        mappings.setProperty("UnauthorizedException", "403");
        r.setExceptionMappings(mappings);  // None by default
        r.setDefaultErrorView("error");    // No default
        r.setExceptionAttribute("ex");     // Default is "exception"
        return r;
    }

    @Bean
    public SimpleCookie rememberMeCookie() {
        SimpleCookie simpleCookie = new SimpleCookie("rememberMe");

        simpleCookie.setPath("/");
        // 30天-30 * 24 * 60 * 60s
        simpleCookie.setMaxAge(259200);
        return simpleCookie;
    }

    @Bean
    public CookieRememberMeManager rememberMeManager() {
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();

        cookieRememberMeManager.setCookie(rememberMeCookie());
        cookieRememberMeManager.setCipherKey("abc.123".getBytes());

        return cookieRememberMeManager;
    }

    @Bean("securityManager")
    public SecurityManager defaultWebSecurityManager(MyShiroRealm realm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(realm);
        securityManager.setRememberMeManager(rememberMeManager());

        return securityManager;
    }

}
```

# 权限、认证信息过滤

这里是实现我们以上需求的关键，即当请求方未登录的时候，就返回未登录`json`信息。这主要是通过继续`AdviceFilter`类，并重载`preHandle`方法实现。

> 这个类允许通过很多方法实现filter中的aop的特点，比如preHandle（前置通知）,postHandle（后置通知，但是在抛异常的情况下可能不执行，）,afterCompletion（最终通知，一定会执行）。


```java
/**
 * 处理未登陆的情况
 */
public class ShiroAuthFilter extends AdviceFilter{

    public static final String error = "未登陆";

    @Override
    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {

        Subject subject = SecurityUtils.getSubject();
        if(subject == null || subject.getPrincipal() == null || !subject.isAuthenticated()){
            // 获取token登陆
            ApiMessage apiMessage = new ApiMessage();
            apiMessage.setSuccess(false);
            apiMessage.setCode(403);
            apiMessage.setMessage(error);
            response.setCharacterEncoding("UTF-8");
            response.setContentType("application/json");
            response.getWriter().write(JSON.toJSONString(apiMessage));
            return false;
        }

        return true;
    }
}

```

通过以上处理，当未登录的时候就会返回`false`，并向`response`流中写入错误信息。