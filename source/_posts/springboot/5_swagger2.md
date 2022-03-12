---
title: Spring Boot Springboot整合Swagger2
author: wangd1
avatar: /img/tux1.jpg
authorLink: wangd1.top
comments: true
tags:
  - Spring Boot
abbrlink: 19684
date: 2019-10-14 21:10:11
photos:
---

## SpringBoot整合Swagger2

### swagger2 是一个规范和完整的框架，用于生成、描述、调用和可视化Restful风格的web服务：

1. 接口文档在线自动生成，文档随接口变动实时更新，节省维护成本
2. 支持在线接口测试，不依赖第三方工具

### 更可以大大的减少沟通成本。

<!--more-->

### 创建项目，添加相关依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 配置Swagger2

创建一个Swagger2的配置类：

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .pathMapping("/")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.wd.springboot09swagger2.controller"))
                .paths(PathSelectors.any())
                .build().apiInfo(new ApiInfoBuilder()
                        .title("SpringBoot整合Swagger2")
                        .description("SpringBoot整合Swagger")
                        .version("1.0")
                        .contact(new Contact("Contact","http://wangd1.com","wangd1@gmail.com"))
                        .license("The Apache License")
                        .licenseUrl("http://www.baidu.com")
                        .build());
    }
}
```

### 

使用@EnableSwagger2注解启用Swagger2，然后配置一个Bean，在这个Bean中，配置映射路径和要扫描的接口的位置，然后配置一下Swagger2文档网站的信息ApiInfo，例如网站的title，网站的描述，联系人的信息，使用的协议等等。

然后启动项目，输入http://localhost:8080/swagger-ui.html看到以下页面就成功了。

![image-20191216193735262](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.37/blogimg/swagger2/image-20191216193735262.png)

### 创建接口

配置完成后，就可以编写一些接口

```java
@RestController
@Api(tags = "用户管理相关接口")
@RequestMapping("/user")
public class UserController {
    @PostMapping("/")
    @ApiOperation("添加用户的接口")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "username", value = "姓名", defaultValue = "wangd1"),
            @ApiImplicitParam(name = "address", value = "地址", defaultValue = "合肥", required = true)}
    )
    public ResponseBean addUser(String username, @RequestParam(required = true) String address) {
        System.out.println(username);
        System.out.println(address);
        return new ResponseBean();
    }

    @GetMapping("/")
    @ApiOperation("根据id查询用户的接口")
    @ApiImplicitParam(name = "id", value = "用户id", defaultValue = "99", required = true)
    public User getUserById(@PathVariable Long id) {
        User user = new User();
        user.setId(id);
        return user;
    }
    @PutMapping("/{id}")
    @ApiOperation("根据id更新用户的接口")
    public User updateUserById(@RequestBody User user) {
        return user;
    }
}
```

其中

@Api表明接口的作用，

@ApiOperation表明方法的作用

@ApiImplicitParam用来配置方法参数的一些含义，也可以设置默认值，如果有多个参数，需要将其放在@ApiImplicitParams注解中。

另外是@ApiImplicitParam中的required仅对swagger2框架有用，脱离了就没用了。所以@RequestParam(required = true)最好不要省略。

配置完后，重启，打开刚刚的页面，可以看到：

![image-20191216194658917](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.37/blogimg/swagger2/image-20191216194658917.png)

![image-20191216194839608](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.37/blogimg/swagger2/image-20191216194839608.png)

![image-20191216194934561](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.37/blogimg/swagger2/image-20191216194934561.png)

一目了然，并且可以通过右上角的Try it out来测试接口。

### swagger-ui.html 中导出的API文件可以在 Swagger Editor（[http://editor.swagger.io/](https://links.jianshu.com/go?to=http%3A%2F%2Feditor.swagger.io%2F)) 中打开














