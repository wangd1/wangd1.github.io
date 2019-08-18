title: Spring Boot Web 静态资源的映射规则
author: wangd1
avatar: '/img/tux1.jpg'
authorLink: wangd1.top
comments: true
date: 2019-7-3 22:12:15
tags: [Spring Boot]
photos:

---

## SpringBoot对静态资源的映射规则

1. 所有/webjars/**，都去classpath:/META-INFO/resources/webjars找资源；

   webjars：以jar包的形式引入静态资源。（[https://www.webjars.org/](https://www.webjars.org/)）

   ```xml
   <!--引用jquery的webjars-->
   <dependency>
       <groupId>org.webjars</groupId>
       <artifactId>jquery</artifactId>
       <version>3.4.1</version>
   </dependency>
   ```

   访问：http://localhost:8085/webjars/jquery/3.4.1/jquery.js

2. "/**" 访问当前项目的任何资源（静态资源文件夹）

   "classpath:/META_INFO/resources/"，

   "classpath:/resources/"，

   "classpath:/static/"，

   "classpath:/public/"，

   "/"：当前项目的根路径，

   欢迎页：静态资源文件夹下的所有index.html

   图标：静态资源文件夹下的favorite.ico

   修改静态资源文件夹路径：

   ```properties
   spring.resources.static-locations=classpath:/hello/,classpath:/abc/
   ```

   