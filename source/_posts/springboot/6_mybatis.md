---
title: Spring Boot Mybatis(注解)、多数据源
author: wangd1
avatar: /img/tux1.jpg
authorLink: wangd1.top
comments: true
tags:
  - Spring Boot
abbrlink: 58111
date: 2019-10-18 20:12:15
photos:
---

## SpringBoot整合Mybatis

### 创建工程，添加依赖

```xml
<!--连接池-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--Mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
 <!--MySql驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

<!--more-->

### 配置属性文件

```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test01?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
```

### 添加一些注解

配置完成后，就可以编写一些Mapper，并搭配注解使用了

```java
public interface UserMapper {    
    @Select("select * from user")    
    List<User> getAll();   
    
    @Results({            
        @Result(property = "id", column = "id"),            
        @Result(property = "username" , column = "name"),            
        @Result(property = "address" , column = "a")    })    
    
    @Select("select id as id, username as u,address as a from user where id=#{id}")    
    User get(Long id);    
    
    @Select("select * from user where username like concat('%',#{name},'%')")    
    List<User> getByName(String name);   
    
    @Insert({"insert into user(username,address) values(#{username},#{address})"})   
    @SelectKey(statement = "select last_insert_id()", keyProperty = "id", before = false, resultType = Long.class)    
    Integer add(User user);    
    
    @Update("update user set username=#{username},address=#{address} where id=#{id}")    
    Integer update(User user);    
    
    @Delete("delete from user where id=#{id}")    
    Integer delete(Long id);}
```

使用注解来写SQL，和XML方式都可以对应上，@Select,@Insert,@Update,@Delete就分别对应<select></select>、<insert></insert>、<update></update>、<delete></delete>，@Results注解类似于XML中的ResultMap映射文件，@SelectKey注解可以实现主键回填的功能，即当数据插入成功后，插入成功的数据id会赋值到user对象的id属性上。

另外还需要配置注解扫描，在Mapper类上使用@Mapper或者在启动类前加上@MapperScan(basePackages = "com.wd.m.mapper")注解，指定Mapper类所在路径。

### 测试

在单元测试中注入UserMapper，然后编写测试类测试。

```java
User user = new User();
user.setUsername("wangd1").setAddress("安徽合肥");
Integer isInsert = userMapper.add(user);
System.out.println(isInsert);
System.out.println(user);
```



## SpringBoot整合Mybatis多数据源：

### 依然创建工程，添加依赖，和上面一样

```xml
<!--连接池-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--Mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
<!--MySql驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 配置多数据源

多加一个数据源：

```properties
spring.datasource.one.url=jdbc:mysql://127.0.0.1:3306/test01?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
spring.datasource.one.username=root
spring.datasource.one.password=root
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource

spring.datasource.two.url=jdbc:mysql://127.0.0.1:3306/test02?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT
spring.datasource.two.username=root
spring.datasource.two.password=root
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource

```

提供两个DataSource：

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.one")
    DataSource dsOne(){
        return DruidDataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.two")
    DataSource dsTwo(){
        return DruidDataSourceBuilder.create().build();
    }

}
```

添加两个Mybatis配置类：

创建MyBatisConfigOne类，首先指明该类是一个配置类，配置类中要扫描的包是com.wd.springboot07mybatis.mapper，即该包下的Mapper接口将操作dsOne中的数据，对应的SqlSessionFactory和SqlSessionTemplate分别是sqlSessionFactory1和sqlSessionTemplate1，在MyBatisConfigOne内部，分别提供SqlSessionFactory和SqlSessionTemplate即可，SqlSessionFactory根据dsOne创建，然后再根据创建好的SqlSessionFactory创建一个SqlSessionTemplate。

```java
@Configuration
@MapperScan(basePackages = "com.wd.springboot07mybatis.mapper",sqlSessionFactoryRef = "sqlSessionFactory1",sqlSessionTemplateRef = "sqlSessionTemplate1")
public class MyBatisConfigOne {
    @Resource(name = "dsOne")
    DataSource dsOne;

    @Bean
    SqlSessionFactory sqlSessionFactory1() {
        SqlSessionFactory sessionFactory = null;
        try {
            SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
            bean.setDataSource(dsOne);
            sessionFactory = bean.getObject();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return sessionFactory;
    }
    @Bean
    SqlSessionTemplate sqlSessionTemplate1() {
        return new SqlSessionTemplate(sqlSessionFactory1());
    }
}
```

```java
@Configuration
@MapperScan(basePackages = "com.wd.springboot07mybatis.mapper2",sqlSessionFactoryRef = "sqlSessionFactory2",sqlSessionTemplateRef = "sqlSessionTemplate2")
public class MyBatisConfigTwo {
    @Resource(name = "dsTwo")
    DataSource dsTwo;

    @Bean
    SqlSessionFactory sqlSessionFactory2() {
        SqlSessionFactory sessionFactory = null;
        try {
            SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
            bean.setDataSource(dsTwo);
            sessionFactory = bean.getObject();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return sessionFactory;
    }
    @Bean
    SqlSessionTemplate sqlSessionTemplate2() {
        return new SqlSessionTemplate(sqlSessionFactory2());
    }
}
```

然后可以创建两个Mapper包，mapper和mapper2

然后测试：通过注入不同的Mapper，就可以操作不同的数据源：

```java
@Autowired
private UserMapper2 userMapper2;
@Autowired
private UserMapper userMapper;

@Testvoid contextLoads() {   
    User user = new User();    
    user.setUsername("wangd1").setAddress("安徽合肥");    
    Integer isInsert = userMapper2.add(user);    
    System.out.println(isInsert);    
    System.out.println(user);    
}
```

### 另外多数据源的整合也可以通过AOP实现，有空再研究一下














