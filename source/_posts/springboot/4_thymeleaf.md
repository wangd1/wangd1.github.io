title: Spring Boot Thymeleaf
author: wangd1
avatar: '/img/tux1.jpg'
authorLink: wangd1.top
comments: true
date: 2019-7-4 20:12:15
tags: [Spring Boot]
photos:

---

## Thymeleaf介绍

> `Thymeleaf`是现代化服务器端的Java模板引擎，不同与其它几种模板的是`Thymeleaf`的语法更加接近HTML，并且具有很高的扩展性。详细资料可以浏览[官网](https://www.thymeleaf.org/)。
>
> 支持无网络环境下运行，可直接运行，这样会加载默认的数据。如果数据返回到页面时，Thymeleaf的标签会动态替换掉静态内容。
>
> SpringBoot官方推荐模板，提供了可选集成模块(`spring-boot-starter-thymeleaf`)，可以快速的实现表单绑定、属性编辑器、国际化等功能。

<!--more-->

## Thymeleaf简单使用

首先创建一个springboot web项目。

然后在pom.xml文件中添加对 thymeleaf 的依赖：

```xml
<!--引入thymeleaf-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

可以在properties文件中对thymeleaf配置，配置项可以在ThymeleafProperties.class文件中找到：

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    private Charset encoding;
    private boolean cache;
    ...
    ...
}
```

创建一个ThymeleafController.java和templates/welcome.html

ThymeleafController：

```java
@GetMapping("/hello")
public String hello(Map<String,Object> map){

    map.put("hello","hello thymeleaf");

    return "welcome";
}
```

welcome.html：

加入 `<html lang="en" xmlns:th="http://www.thymeleaf.org">`会有语法提示。

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Welcome</h1>
    <p th:text="${hello}">你好</p>
</body>
</html>
```

运行，浏览器输入 [http://localhost:8085/hello](http://localhost:8085/hello)可以看到效果：

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.36/blogimg/thymeleaf_img/1566312042483.png)

如果直接运行这个html，则会显示默认值：

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.36/blogimg/thymeleaf_img/1566312086497.png)

## Thymeleaf语法

可以参考 [https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.pdf](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.pdf)














