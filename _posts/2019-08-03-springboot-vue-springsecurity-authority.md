---
layout: post
title: SpringBoot+Vue实战（三）- 集成SpringSecurity实现用户权限限制功能
date: 2019-08-03 20:00:00
tags: [Spring Boot, Vue]
categories: [SpringBoot+Vue实战]
baseline: 
artileno: 20190803_01
---

上一篇文章我们实现了，用户功能，如果我们限制不同用户访问不同的服务地址，该怎么办呢？

我们先来规划下，要实现什么样的权限划分，上节我们在数据库中 创建了三个用户 admin root testuser，我们来给他们增加角色

| 用户 | 角色 |
| --- | --- |
|admin | admin |
| root | admin |
| testuser | user |

* 数据库中 角色名 必须增加上 ROLE_ 前缀

资源和角色对应关系

| 资源 | 角色 |
| --- | --- |
| /admin/hello | admin |
| /user/hello | admin |
| /user/hello | user |

来解决这个问题之前，老样子，先做些准备 

* 0x01 准备数据库表及数据 上篇增加了用户表 本篇还要新增 4⃣️张表 角色表 用户角色关系表 资源表 资源角色关系表（在这里 我把访问的服务地址称为资源）

```
# 角色表
CREATE TABLE `sys_role` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `no` varchar(64) NOT NULL,
  `name` varchar(64) NOT NULL,
  `name_zh` varchar(64) NOT NULL,
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `creator` varchar(64) NOT NULL,
  `modify_at` timestamp NULL DEFAULT NULL,
  `modifier` varchar(64) DEFAULT NULL,
  `remark` varchar(256) DEFAULT NULL,
  `is_del` tinyint(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `id` (`id`),
  UNIQUE KEY `no_UNIQUE` (`no`)
) 
# 初始化数据


# 用户角色关系表
CREATE TABLE `sys_user_role` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `no` varchar(64) NOT NULL,
  `user_no` varchar(64) NOT NULL,
  `role_no` varchar(64) NOT NULL,
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `creator` varchar(64) NOT NULL,
  `modify_at` timestamp NULL DEFAULT NULL,
  `modifier` varchar(64) DEFAULT NULL,
  `remark` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `id` (`id`)
)
# 初始化数据

# 资源表 
CREATE TABLE `sys_resource` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `no` varchar(64) NOT NULL,
  `url_pattern` varchar(64) NOT NULL,
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `creator` varchar(64) NOT NULL,
  `modify_at` timestamp NULL DEFAULT NULL,
  `modifier` varchar(64) DEFAULT NULL,
  `remark` varchar(256) DEFAULT NULL,
  `is_del` tinyint(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `id` (`id`),
  UNIQUE KEY `no` (`no`),
  UNIQUE KEY `id_UNIQUE` (`id`)
) 

# 初始化数据

# 资源角色关系表
CREATE TABLE `sys_resource_role` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `no` varchar(64) NOT NULL,
  `resource_no` varchar(64) NOT NULL,
  `role_no` varchar(64) NOT NULL,
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `creator` varchar(64) NOT NULL,
  `modify_at` timestamp NULL DEFAULT NULL,
  `modifier` varchar(64) DEFAULT NULL,
  `remark` varchar(256) DEFAULT NULL,
  `is_del` tinyint(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `id` (`id`),
  UNIQUE KEY `no_UNIQUE` (`no`)
)

# 初始化数据
```

* 0x02 为了验证权限 我们调整下 HelloController 代码如下

```
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello Cobra!";
    }

    @RequestMapping("/admin/hello")
    public String admin_hello() {
        return "Hello Admin Cobra!";
    }

    @RequestMapping("/user/hello")
    public String user_hello() {
        return "Hello User Cobra!";
    }
}
```

使用 MybatisGenerator 生成的表的 Entity Mapper(自己个Google))

修改下 User 实体，增加上权限属性 roles， 并修改下 getAuthorities 方法，获取当前用户对象所具有的角色信息

```
    private List<Role> roles;

    // 获取当前用户对象所具有的角色信息
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<SimpleGrantedAuthority>();
        if (roles != null){
            for (Role role :roles){
                authorities.add(new SimpleGrantedAuthority(role.getName()));
            }
        }
        return authorities;
    }
```

修改 UserService 类，从数据库中查询出用户的角色信息 

```
@Service
@Slf4j
public class UserService implements UserDetailsService {
    @Autowired
    UserMapper userMapper;
    @Autowired
    RoleMapper roleMapper;
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        User user = userMapper.loadUserByUsername(s);
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在!");
        }
        user.setRoles(roleMapper.getUserRolesByUserNo(user.getNo()));
        return user;
    }
}
```

至此，当前用户 已经拥有了角色，下面我们将使用两种方式 来实现角色的权限分配

* 静态权限分配
* 动态权限分配 

#### 静态权限分配 

重写 WebSecurityConfig 中的 configure(HttpSecurity http) 方法

```
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("admin")
                .antMatchers("/user/**").hasRole("user")
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginProcessingUrl("/login").permitAll()
                .and()
                .csrf().disable();
    }
```        

来解释下 
- 用户 有 admin 角色 可以访问 /admin/** 的路径 有 user 角色 可以访问 /user/** 的路径 
- anyRequest().authenticated() 除去匹配上的路径 其他的路径 登录后 均可访问

#### 动态权限分配 

在一些情况下 用户的权限可能发生变化 需要动态来调整权限分配 

需要自定义 FilterInvocationSecurityMetadataSource 接口，默认实现类是 DefaultFilterInvocationSecurityMetadataSource 可以参考一下，来实现自己的类

实现接口中 getAttributes(Object o) 方法 来判断服务地址需要用户拥有哪些角色才可以访问

```
@Configuration
public class CobraFilterInvocationSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {
    AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Autowired
    ResourceService resourceService;

    @Override
    public Collection<ConfigAttribute> getAttributes(Object o) throws IllegalArgumentException {
        String requestUrl = ((FilterInvocation) o).getRequestUrl();
        List<Resource> resources = resourceService.resourceList();
        for (Resource resource : resources) {
            List<Role> roles = resourceService.resourceRoles(resource.getNo());
            if (antPathMatcher.match(resource.getUrlPattern(), requestUrl) && roles.size() > 0) {
                String[] roleArr = new String[roles.size()];
                for (int i = 0; i < roles.size(); i++) {
                    roleArr[i] = roles.get(i).getName();
                }
                return SecurityConfig.createList(roleArr);
            }
        }
        return SecurityConfig.createList("ROLE_LOGIN");
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return FilterInvocation.class.isAssignableFrom(aClass);
    }
}
```

实现接口中 AccessDecisionManager 的 decide(Authentication authentication, Object o, Collection<ConfigAttribute> collection)  方法 来判断当前用户的角色是否包含服务地址的角色权限

主要代码如下：

```
@Configuration
public class CobraAccessDecisionManager implements AccessDecisionManager {
    @Override
    public void decide(Authentication authentication, Object o, Collection<ConfigAttribute> collection) throws AccessDeniedException, InsufficientAuthenticationException {
        Collection<? extends GrantedAuthority> auths = authentication.getAuthorities();
        for (ConfigAttribute ca : collection) {
            if ("ROLE_LOGIN".equals(ca.getAttribute()) && authentication instanceof UsernamePasswordAuthenticationToken) {
                return;
            }
            for (GrantedAuthority ga : auths) {
                if (ca.getAttribute().equals(ga.getAuthority())) {
                    return;
                }
            }
        }
        throw new AccessDeniedException("权限不足");
    }

    @Override
    public boolean supports(ConfigAttribute configAttribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}
```

接下来我们再来调整下 WebSecurityConfig 中的 configure(HttpSecurity http) 方法

```
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    UserService userService;

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(10);// 密码强度 10
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService);
    }

    // 设置 路径和角色的匹配关系
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {

                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(O o) {
                        o.setSecurityMetadataSource(cfisms());
                        o.setAccessDecisionManager(new CobraAccessDecisionManager());
                        return o;
                    }
                })
                .and()
                .formLogin().loginProcessingUrl("/login").permitAll()
                .and()
                .csrf().disable();
    }

    // 通过这种方式 可以注入@Autowire
    @Bean
    CobraFilterInvocationSecurityMetadataSource cfisms() {
        return new CobraFilterInvocationSecurityMetadataSource();
    }
}
```                                                                                                                    
好了 至此我们完成了 权限的动态分配。

来验证一下 访问 http://127.0.0.1:9999/admin/hello 跳转到登录页面

![登录页面]({{site.url}}/assets/20190801_02_03.png)     

使用 testuser 用户登录成功 访问 /admin/hello 没有权限

![没有权限]({{site.url}}/assets/20190803_01_01.png)

访问 /user/hello 成功

![有权限]({{site.url}}/assets/20190803_01_02.png)