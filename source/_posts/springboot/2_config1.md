---
title: Spring Boot 配置文件1
author: wangd1
avatar: /img/tux1.jpg
authorLink: wangd1.top
comments: true
tags:
  - Spring Boot
abbrlink: 61325
date: 2019-07-02 20:12:15
photos:
---

Spring Boot 使用一个全局的配置文件，文件名是固定的：application.properties或application.yml。配置文件要是想被Spring Boot自动加载，需要放置到指定的位置：src/main/resource目录下，一般自定义的配置文件也位于此目录之下。

<!--more-->

```
优先级：properties > yml
```

配置文件的作用是：修改Spring Boot的自动配置的默认值。

## 1. properties 文件格式

```yaml
server.port=8088
person.last-name=zhangsna
person.age=19
person.birth=2019/10/1
person.boss=true
person.maps.k1=v1
person.maps.k2=v2
person.lists=a,b,c
person.dog.name=dog
person.dog.age=2
```

## 2. yml文件格式

**基本语法**

> K:(空格必须有)V（表示一个键值对，并且属性和值大小写敏感）

以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是一个层级的。

如下：

```yaml
server:
  port: 8088
```

**值的写法**

1. 字面量：普通的值（数字，字符串，布尔）

值可以直接来写，且字符串不用加引号；

如果加引号：

```yaml
name0: "zhangsan \n lisi" #使用双引号会输出  zhangsan 换行 lisi

name1: 'zhangsan \n lisi' #使用单引号会输出  zhangsan \n lisi
```

2. 对象、Map：

键值对写法，如

```yaml
person:
  lastName: zhangsan
  age: 20
```

行内写法，如

```yaml
maps: {k1: v1,k2: v2}
```

3. 数组（List等）：

用换横线（-）表示一个数组元素，如

```yaml
lists:
    - lisi
    - wangwu
    - zhaoliu
```

行内写法，如

```yaml
lists: [lisi,wangwu,zhaoliu]
```

**演示**

```yaml
server:
  port: 8088

person:
  lastName: zhangsan
  age: 20
  boss: true
  birth: 2019/10/1
  maps: {k1: v1,k2: v2}
  lists:
    - lisi
    - wangwu
    - zhaoliu
  dog:
    name: 笨笨
    age: 2
```


## 3. 获取配置文件的值(@ConfigurationProperties和@Component方式)

创建了一个Person类和Dog类，加入@ConfigurationProperties和@Component注解

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中，
 * @ConfigurationProperties 是告诉SpringBoot将本类中的所有属性和配置文件中的相关配置进行绑定
 * prefix = "person" 与配置文件中那个下面的属性进行一一映射
 *
 * 只有这个组件是容器的中的组件，才可以使用这个@ConfigurationProperties注解的功能
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
    ...get set toString()
}

public class Dog {
    private String name;
    private Integer age;
    ...get set toString()
}
```

加入提供的maven依赖，可以在编写yml文件时有提示。

```xml
<!-- 导入配置文件处理器，配置文件进行绑定时会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.33/blogimg/config/1565872626559.png)

然后编写单元测试，注入Person，然后打印配置文件里的值；

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot02ConfigApplicationTests {

    @Autowired
    Person mPerson;
    
    @Test
    public void contextLoads() {
        System.out.println(mPerson);
    }
}
```

运行单元测试可以看到结果：

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.33/blogimg/config/1565872871411.png)

如果使用的是properties文件且有中文的话，打印出来会乱码（idea的properties配置文件编码是utf-8），需要配置一下就不会乱码了。

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.33/blogimg/config/1565873726839.png)
## 4. 获取配置文件的值(@Value方式)

@Value可以获取的值有 **字面量**（字符串等）、**${key}**（从环境变量、配置文件中获取值）、**#{SpEL}**

如：

```java
@Component
//@ConfigurationProperties(prefix = "person")
public class Person {

    @Value("${person.lastName}")
    private String lastName;
    @Value("#{10+10}")
    private Integer age;
    @Value("false")
    private Boolean boss;
    ...
}
```
打印

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.33/blogimg/config/1565874363695.png)
## 5. @Value获取值和@ConfigurationProperties获取值比较

|                                        |                            @Value                            |     @ConfigurationProperties     |
| :------------------------------------: | :----------------------------------------------------------: | :------------------------------: |
|                  功能                  |                        需一个一个指定                        |     可批量注入配置文件的属性     |
|       松散绑定（松散语法，如下）       | 不支持(@Value里的值要与配置文件中的一致或是使用配置文件里提示的值) |               支持               |
|                  SpEL                  |                             支持                             | 不支持（配置文件中不支持表达式） |
| JSP303数据校验（如@Validated，@Email） |                            不支持                            |               支持               |
|           复杂类型封装(Map)            |                            不支持                            |               支持               |

属性名匹配规则

- person.lastName：标准方式
- person.last-name：大写用-
- person.last_name：大写用_
- PERSON_LAST_NAME

配置文件是yml或properties都可以获取到值。

所以，

如果说只是在某个业务逻辑中需要用到配置文件中的某项值，用@Value比较灵活

如果说专门写了一个javaBean来和配置文件相映射，就直接使用@ConfigurationProperties



未完...



