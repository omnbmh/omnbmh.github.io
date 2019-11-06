---
layout: post
title: SpringBoot+Vue实战（二）- 集成SpringSecurity实现用户登录功能
date: 2019-08-01 19:50:00
tags: [Spring Boot, Vue]
categories: [SpringBoot+Vue实战]
baseline: 
artileno: 20190801_02
---

从这篇文章开始，我们一点一点的来完善我们的项目。

一个后管理系统，肯定需要区分不同的角色，来分配不同的权限，那我们就是用SpringSecurity来实现用户登录功能。

本篇用4种不同的方式来实现用户登录

- 基于默认用户的认证
- 基于配置用户的认证
- 基于内存用户的认证
- 基于数据库用户的认证

* 0x01 添加 Maven 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

* 0x02 新增一个 Controller，用来验证登录后的效果

```
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello Cobra!";
    }
}
```

* 0x03 SpringBoot 的启动文件 CobraApplication.java

```
@SpringBootApplication
public class CobraApplication {
    public static void main(String[] args) {
        SpringApplication.run(CobraApplication.class, args);
    }
}
```

* 0x04 接下来，我们来尝试使用文章开头说的几种方式来实现登录

#### 基于默认用户的认证

基于默认用户，我们什么也不用做，直接启动项目 Run CobraApplication

```
# 日志中 找一下密码 
INFO 23491 --- [           main] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: 199feb63-3f5b-4288-9f5a-ccd07d9e54f4
```

密码是随机的，每次启动生成一个

访问 http://127.0.0.1:9999/hello 跳转到登录页面 登录页是 SpringSecurity 提供的默认登录页面，默认用户名 user 密码是日志中的打印的密码

![登录页面]({{site.url}}/assets/20190801_02_01.png)

登录成功，跳转到欢迎页面

![登录页面]({{site.url}}/assets/20190801_02_02.png)

#### 基于配置文件用户的认证

我们也可以在配置文件中指定 用户名和密码 如下

```
#在 application.properties 中添加
spring.security.user.name=root
spring.security.user.password=123456
spring.security.user.roles=admin
```

访问 http://127.0.0.1:9999/hello 跳转到登录页面

![登录页面]({{site.url}}/assets/20190801_02_01.png)

登录成功，跳转到欢迎页面

![登录页面]({{site.url}}/assets/20190801_02_02.png)

## 基于内存用户的认证

不管是默认用户，还是配置用户，都是指定了一个用户，想要由多个用户怎么办？

我们可以重写 WebSecurityConfigurerAdapter 的 configure(AuthenticationManagerBuilder auth) 方法来实现

```
# 新增 WebSecurityConfig
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("admin").password("123456").roles("admin", "user")
                .and()
                .withUser("testuser").password("123456").roles("user");
    }
}
```

代码解析:

* Bean passwordEncoder 方法定义了 用户密码的加密方式 代码方便配置用户使用了不加密
* 新增了 admin testuser 两个用户
* 内存用户配置后，在配置文件中的用户会失效

访问 http://127.0.0.1:9999/hello 跳转到登录页面

![登录页面]({{site.url}}/assets/20190801_02_01.png)

登录成功，跳转到欢迎页面

![登录页面]({{site.url}}/assets/20190801_02_02.png)


## 基于数据库用户的认证

真实的项目中用户不是固定的，通过配置的方式添加用户不太现实，接下来基于mysql数据库实现认证。

初始化数据库 

```
# 建表语句
CREATE TABLE `sys_user` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `no` varchar(64) NOT NULL,
  `username` varchar(64) NOT NULL,
  `passwd` varchar(256) NOT NULL,
  `is_enable` tinyint(1) NOT NULL DEFAULT '0',
  `is_locked` tinyint(1) NOT NULL DEFAULT '0',
  `create_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `creator` varchar(64) NOT NULL,
  `modify_at` timestamp NULL DEFAULT NULL,
  `modifier` varchar(64) DEFAULT NULL,
  `remark` varchar(256) DEFAULT NULL,
  `is_del` tinyint(1) NOT NULL DEFAULT '0' COMMENT '0=未删除 1=删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `id` (`id`),
  UNIQUE KEY `id_UNIQUE` (`id`),
  UNIQUE KEY `no_UNIQUE` (`no`),
  UNIQUE KEY `username_UNIQUE` (`username`)
)

# 初始化用户 admin root testuser
INSERT INTO `sys_user` (`id`,`no`,`username`,`passwd`,`is_enable`,`is_locked`,`create_at`,`creator`,`modify_at`,`modifier`,`remark`,`is_del`) VALUES (3,'2019110415503872900001','admin','$2a$10$BdiHZbGdQ/sfHf0SeJAFwuaaxvM/.eNyoUeeKZMYNVH4tiXpQm7Jm',1,0,'2019-11-04 15:50:39','system',NULL,NULL,NULL,0);
INSERT INTO `sys_user` (`id`,`no`,`username`,`passwd`,`is_enable`,`is_locked`,`create_at`,`creator`,`modify_at`,`modifier`,`remark`,`is_del`) VALUES (4,'2019110415523464100001','root','$2a$10$1lunk4Gpe.ZdE7Fzxw3LgeVZETkbWZkYBFZCNKIeADlM7AaDdmING',1,0,'2019-11-04 15:52:35','system',NULL,NULL,NULL,0);
INSERT INTO `sys_user` (`id`,`no`,`username`,`passwd`,`is_enable`,`is_locked`,`create_at`,`creator`,`modify_at`,`modifier`,`remark`,`is_del`) VALUES (5,'2019110415535120200001','testuser','$2a$10$QrrlMLxubZ6cGMcxVZUZherge6uh2ZWsvDDBcq/39LOfnIRATy9wK',1,0,'2019-11-04 15:53:52','system',NULL,NULL,NULL,0);

```

调整 Maven 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

* 使用了 Druid 连接池

配置文件 application.properties 加上 mysql 数据库的配置

```
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/cobra?useUnicode=true&characterEncoding=utf-8
spring.datasource.username=cobra
spring.datasource.password=123456
```
使用 MybatisGenerator 生成的表的 Entity Mapper(自己个Google))

需要在配置文件中  application.properties 指定 Mapper 的位置

```
mybatis.mapper-locations=classpath:mapper/*.xml
```

调整 User 实体，实现 UserDetails 接口 重写以下 7 个方法

| 方法名 | 解释|
| --- | --- |
| getAuthorities | 获取当前用户的角色权限信息 |
| getPassword | 获取当前用户密码 |
| getUsername | 获取当前用户名 |
| isAccountNonExpired | 当前用户是否过期 |
| isAccountNonLocked | 当前用户是否锁定 |
| isCredentialsNonExpired | 当前用户密码是否过期 |
| isEnabled | 当前用户是否启用 |

主要代码如下 

```
public class User implements UserDetails {
 
    ...

    // 获取当前用户对象所具有的角色信息
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<SimpleGrantedAuthority> authorities = new ArrayList<SimpleGrantedAuthority>();
        return authorities;
    }

    @Override
    public String getPassword() {
        return passwd;
    }

    @Override
    public String getUsername() {
        return username;
    }

    // 账户不过期
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    // 账户是否锁定
    @Override
    public boolean isAccountNonLocked() {
        return !isLocked;
    }

    // 证书不过期
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    // 是否启用
    @Override
    public boolean isEnabled() {
        return isEnable;
    }

    ...
}
```

创建 UserService 实现 UserDetailService 接口的 loadUserByUsername 方法，来从数据库中获取用户。

主要代码如下：

```
@Service
public class UserService implements UserDetailsService {
    @Autowired
    UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        User user = userMapper.loadUserByUsername(s);
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在!");
        }
        return user;
    }
}
```

* Mapper 中查询用户方法 loadUserByUsername 可自行实现

最后，我们调整下 WebSecurityConfig 改一下 configure(AuthenticationManagerBuilder auth) 方法 

主要代码如下：

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
}
```

* 增加Bean passwordEncoder 设置了用户密码的加密方式，看下添加用户的 sql，里面的原始密码是 123456
* 设置自己实现的 userService

访问 http://127.0.0.1:9999/hello 跳转到登录页面

![登录页面]({{site.url}}/assets/20190801_02_01.png)

我们使用 testuser 用户登录成功，跳转到欢迎页面

![登录页面]({{site.url}}/assets/20190801_02_02.png)