title: Spring Boot 学习 01：HelloWorld
author: wangd1
avatar: '/img/tux1.jpg'
authorLink: wangd1.top
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
date: 2019-7-1 21:12:15
tags: [Spring Boot]
photos:

---

## 1.简介

Spring Boot的设计目的就是简化Spring应用的开发，开启了各种自动装配，约定大于配置，去繁从简，引入相关的依赖就可以迅速搭建起一个web工程。

<!--more-->

### 优点

#### 快速创建独立运行的Spring项目以及与主流框架集成

#### 使用嵌入式的Servlet容器，应用无需打成war包

#### starters自动依赖与版本控制

#### 大量的自动配置，简化开发，也可以修改默认值

#### 无需配置XML文件，无代码生成，开箱即用

#### 准生产环境的运行时应用监控

#### 与云计算天然集成






## 2.构建工程

### 使用IDEA构建Spring Boot工程

打开IDEA，File->New Project/New Module->Spring Initializr，下一步，填写好Group，Artifact，下一步，勾选Spring Web Starter开启web功能，然后输入项目名称，Finish。

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565792271519.png)

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565792316291.png)

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565792333087.png)



### 工程目录（controller是自己创建的）

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565791616788.png)

其中：

pom 文件为基本的依赖管理文件
resouces 资源文件
statics 静态资源
templates 模板资源
application.yml / application.properties 配置文件
SpringbootXXXApplication 程序的入口。

### 演示

创建一个controller

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "hello world";
    }
}
```

其中 

@RestController 是 @ResponseBody和@Controller两个注解的结合，表示方法返回的数据直接写给浏览器，如果是对象就转为json数据。

@GetMapping 注解就是@RequestMapping(method = RequestMethod.GET)注解的缩写

然后启动SpringBoot01HelloworldApplication的main方法，在浏览器输入[localhost:8080/hello](localhost:8080/hello)

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565792578340.png)

### 打包

点击IDEA工程的右侧 Maven Project，找到对应项目，依次执行clean，package

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565792684277.png)

可以看到jar包路径

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565792740191.png)

命令行定位到jar包目录，执行java -jar xxx.jar，然后在浏览器输入[localhost:8080/hello](localhost:8080/hello)可以看到同样效果

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.32/blogimg/helloworld/1565792892418.png)





