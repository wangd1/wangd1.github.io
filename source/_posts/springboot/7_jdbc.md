---
title: Spring Boot JdbcTemplate多数据源
author: wangd1
avatar: /img/tux1.jpg
authorLink: wangd1.top
comments: true
tags:
  - Spring Boot
abbrlink: 691
date: 2020-09-05 10:12:15
photos:
---

### Spring JdbcTemplate是spring对jdbc的封装，省去了自己管理数据库资源的麻烦，使得Jdbc更容易使用。

### Srping JdbcTemplate配置druid多数据源时，主要需要在创建JdbcTemplate时为其分配不同的数据源，在访问不同数据库的时候，使用不用的JdbcTemplate即可。



<!--more-->

>引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
	<version>1.1.18</version>
</dependency>
```

>配置多数据源

```yml
server:
  port: 8088
spring:
  datasource:
    druid:
      #数据库访问配置
      #数据源1
      one:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/test01?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&serverTimezone=GMT%2B8
        username: root
        password: 8023love
      #数据源2
      two:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/test02?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&serverTimezone=GMT%2B8
        username: root
        password: 8023love

      # 连接池配置
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 连接等待超时时间
      max-wait: 30000
      # 配置检测可以关闭的空闲连接间隔时间
      time-between-eviction-runs-millis: 60000
      # 配置连接在池中的最小生存时间
      min-evictable-idle-time-millis: 300000
      validation-query: select '1' from dual
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      # 打开PSCache，并且指定每个连接上PSCache的大小
      pool-prepared-statements: true
      max-open-prepared-statements: 20
      max-pool-prepared-statement-per-connection-size: 20
      # 配置监控统计拦截的filters, 去掉后监控界面sql无法统计, 'wall'用于防火墙
      filters: stat,wall
      # Spring监控AOP切入点，如x.y.z.service.*,配置多个英文逗号分隔
      aop-patterns: com.wangd1.jdbc.servie.*

      # WebStatFilter配置
      web-stat-filter:
        enabled: true
        # 添加过滤规则
        url-pattern: /*
        # 忽略过滤的格式
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      # StatViewServlet配置
      stat-view-servlet:
        enabled: true
        # 访问路径为/druid时，跳转到StatViewServlet
        url-pattern: /druid/*
        # 是否能够重置数据
        reset-enable: false
        # 需要账号密码才能访问控制台
        login-username: druid
        login-password: druid123
        # IP白名单
        # allow: 127.0.0.1
        #　IP黑名单（共同存在时，deny优先于allow）
        # deny: 192.168.1.218

      # 配置StatFilter
      filter:
        stat:
          log-slow-sql: true
```

创建一个数据源配置类；

```java
@Configuration
public class DataSourceConfig {

    @Bean("dataSourceOne")
    @ConfigurationProperties("spring.datasource.druid.one")
    public DataSource mDataSourceOne(){
        return DruidDataSourceBuilder.create().build();
    }

    @Bean("dataSourceTwo")
    @ConfigurationProperties("spring.datasource.druid.two")
    public DataSource mDataSourceTwo(){
        return DruidDataSourceBuilder.create().build();
    }

    @Primary
    @Bean("jdbcTemplateOne")
    public JdbcTemplate mJdbcTemplateOne(@Qualifier("dataSourceOne") DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

    @Bean("jdbcTemplateTwo")
    public JdbcTemplate mJdbcTemplateTwo(@Qualifier("dataSourceTwo") DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
}
```

上面创建了dataSourceOne和dataSourceOne两个数据源，并根据这两个数据源创建了两个jdbctemplate。

@Primary意思是如果有很多相同的bean，优先使用用了@Primary的bean。

配置多数据源的时候，势必有一个主数据源，不然在不使用@Qualifier("jdbcTemplateOne")注入jdbcTemplate的时候，会报错，因为会匹配到两个jdbcTemplate。

使用@Primary之后，可以不用@Qualifier注解，使用其他数据源的时候再使用@Qualifier。

> 测试

创建表并添加数据

test01.user：

![image-20200908142046423](https://raw.githubusercontent.com/wangd1/cdn/master/image/image-20200908142046423.png)

test02.user：

![image-20200908142100871](https://raw.githubusercontent.com/wangd1/cdn/master/image/image-20200908142100871.png)

然后创建dao和service

IUserOneDao：

```java
public interface IUserOneDao {
    List<Map<String,Object>> getAllUser();
}
```

实现：注入了jdbcTemplateOne

```java
@Repository
public class UserOneDaoImpl implements IUserOneDao {
    @Autowired
    @Qualifier("jdbcTemplateOne")
    private JdbcTemplate mJdbcTemplate;

    @Override
    public List<Map<String, Object>> getAllUser() {
        return mJdbcTemplate.queryForList("select * from user");
    }
}
```

IUserTwoDao：

```java
public interface IUserTwoDao {
    List<Map<String,Object>> getAllUser();
}
```

实现：注入了jdbcTemplateTwo

```java
@Repository
public class UserTwoDaoImpl implements IUserTwoDao {
    @Autowired
    @Qualifier("jdbcTemplateTwo")
    private JdbcTemplate mJdbcTemplate;

    @Override
    public List<Map<String, Object>> getAllUser() {
        return mJdbcTemplate.queryForList("select * from user");
    }
}
```

IUserService：

```java
public interface IUserService {
    List<Map<String,Object>> getAllUserFromOne();
    List<Map<String,Object>> getAllUserFromTwo();
}
```

实现：

```java
@Service
public class UserServiceImpl implements IUserService {

    @Autowired
    private IUserOneDao userOneDao;
    @Autowired
    private IUserTwoDao userTwoDao;

    @Override
    public List<Map<String, Object>> getAllUserFromOne() {
        return userOneDao.getAllUser();
    }

    @Override
    public List<Map<String, Object>> getAllUserFromTwo() {
        return userTwoDao.getAllUser();
    }
}
```

然后UserController：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private IUserService mUserService;

    @GetMapping("/allOne")
    public List<Map<String,Object>> getAllUserOne(){
        return mUserService.getAllUserFromOne();
    }

    @GetMapping("/allTwo")
    public List<Map<String,Object>> getAllUserTwo(){
        return mUserService.getAllUserFromTwo();
    }

}
```

最后启动项目，访问 [http://localhost:8088/user/allOne](http://localhost:8088/user/allOne)

![image-20200908143033057](https://raw.githubusercontent.com/wangd1/cdn/master/image/image-20200908143033057.png)

[http://localhost:8088/user/allTwo](http://localhost:8088/user/allTwo)

![image-20200908143105803](https://raw.githubusercontent.com/wangd1/cdn/master/image/image-20200908143105803.png)

> 完