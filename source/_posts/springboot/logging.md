title: Spring Boot 学习 03：日志配置
author: wangd1
avatar: '/img/tux1.jpg'
authorLink: wangd1.top
comments: true
date: 2019-7-3 20:12:15
tags: [Spring Boot]
photos:

------

常见的框架日志有：log4j，log4j2，logback，common-logging(JCL)，java-util-logging(JUL)和slf4j等。

而日志框架主要分两类：

- 日志门面（抽象）如：jboss-logging，slf4j，JCL
- 日志实现（实现）如：log4j，log4j2，logback，JUL

对于日志实现，JUL实现简陋，很多地方被开发者吐槽，所以排除；log4j和logback是同一个作者开发的，且logback是log4j的升级版，比log4j会更好点，排除log4j；log4j2不是log4j的升级版，因为太过于优秀，部分框架对其的支持有限，所以排除。

所以日志实现选中logback。

对于日志门面，jboss-logging很久未更新，而slf4j的作者就是log4j和logback的作者，所以选用slf4j和logback会比较合得来....

所以日志门面选用slf4j。并且在springboot中也是使用的  slf4j + logback。

<!--more-->

### 简单使用

首先导入slf4j的jar包和logback的jar包

在开发的时候，日志记录方法的调用不应该直接调用日志的实现类，而是调用日志抽象层里面的方法。

参考官方文档：[https://www.slf4j.org/manual.html](https://www.slf4j.org/manual.html)

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

### 如何让系统中的所有的日志都统一到slf4j

在系统中，自己使用了slf4j+logback，但是Spring默认的是commons-logging；如果集成Hibernate，他又自带了jboss-logging；如果集成mybatis，他又有自己的日志系统。

想统一日志框架，都使用slf4j输出日志，可以参考slf4j官网 [https://www.slf4j.org/legacy.html](https://www.slf4j.org/legacy.html)

![legacy](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/legacy.png)

slf4j提供了解决方案：

1. 将系统中的其他日志框架排除出去（删除其他日志框架的jar包，如：commons-logging日志的jar包等）；

2. 然后使用中间包来替代原有的日志框架（如上图的jcl-over-slf4j.jar replaces commons-logging.jar，log4j-over-slf4j.jar replaces log4j.jar等）；

3. 导入slf4j其他日志包的实现；

   那些中间包实际上提供了原有包的功能，但是也对slf4j进行了实现。就像log4j-to-slf4j.jar下面的路径还是和log4j一样，但是里面的日志记录的功能使用的是slf4j：

   ![1566057101545](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566057101545.png)

### Spring Boot的日志关系

新建一个springboot项目，然后打开pom文件，查看依赖（右键>Diagrams>Show...）

看到SpringBoot依赖spring-boot-starter

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566056351946.png)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.1.7.RELEASE</version>
    <scope>compile</scope>
</dependency>
```

而spring-boot-starter依赖spring-boot-starter-logging，所以springboot使用它来做日志功能。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
    <version>2.1.7.RELEASE</version>
    <scope>compile</scope>
</dependency>
```

看一下spring-boot-starter-logging：

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566056814856.png)

所以springboot底层是使用slf4j+logback的方式进行日志记录的，并把其他日志替换成了slf4j；

### [使用和配置](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-logging.html#boot-features-logging-format)

1. 修改日志级别

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot03LoggingApplicationTests {
    Logger logger = LoggerFactory.getLogger(SpringBoot03LoggingApplicationTests.class);
    @Test
    public void contextLoads() {
        //日记级别：
        //从低到高：trace<debug<info<warn>error
        //可以调整输出的日志级别；日志就只在这个级别和之后的高级别生效
        logger.trace("这是trace日志");
        logger.debug("这是debug日志");  //调试日志
        //springboot默认的日志级别是info,所以运行时只会打印info级别和高于info级别的日志信息
        logger.info("这是info日志");  //
        logger.warn("这是warn日志");  //警告日志
        logger.error("这是error日志"); //错误日志
    }
}

```

所以打印是：

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566058398543.png)

但是可以通过properties等配置文件修改日志级别：

```properties
logging.level.com.wangd = trace
```

修改指定位置文件的日志级别后打印如下：

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566058593348.png)

2. 生成日志文件

   指定logging.file

```properties
logging.level.com.wangd = trace

# 不指定路径和文件的情况下，只在控制台打印日志信息
# 若指定了logging.file,而无论是否指定logging.path,都只在指定文件中保存日志信息,而不会在路径中保存日志
# 对于logging.path,若指定了目录，则会在这个目录中自动生成spring.log保存日志信息
#logging.path=/spring/log
logging.file=spring.log
```

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566059238679.png)

指定logging.path=/spring/log

```properties
logging.level.com.wangd = trace

# 不指定路径和文件的情况下，只在控制台打印日志信息
# 若指定了logging.file,而无论是否指定logging.path,都只在指定文件中保存日志信息,而不会在路径中保存日志
# 对于logging.path,若指定了目录，则会在当前磁盘的这个目录中自动生成spring.log保存日志信息
logging.path=/spring/log
#logging.file=spring.log
```

![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566059371896.png)

3. 修改日志输出格式

   ```properties
   # 日志输出格式:
   # %d:日期时间
   # %thread:线程名
   # %-5level:级别从左显示5个字符宽度
   # %logger{50}:表示logger名字最长50个字符,否则按照句点切割
   # %msg:日志消息
   # %n:换行符
   
   # 在控制台输出日志的格式
   logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-2level %logger{5} - %msg%n
   # 在日志文件中输出日志的格式
   logging.pattern.file=%d{yyyy-MM-dd} === [%thread] %-5level %logger{50} - %msg%n
   ```

4. 使用@Slf4j注解

   在使用类名的时候，每个类都需要自己编写
   
   ```java
   Logger mLogger = LoggerFactory.getLogger(XXX.class);
   ```
   
   会很不方便，所以可以通过一个注解（@Slf4j）来帮我们实现，该注解就可以帮我们自动创建一个 log对象。
   
   但是 如果使用 @Slf4j 注解 需要两点，
   
   1. 安装lombok插件，参考idea安装插件
   
   2. 添加 lombok的依赖
   
   ```xml
   <!--添加lombok依赖-->
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
   </dependency>
   ```
   
   就可以使用@Slf4j注解了。
   
   ```java
   @RunWith(SpringRunner.class)
   @SpringBootTest
   @Slf4j
   public class SpringBoot03LoggingApplicationTests {
       @Test
       public void contextLoads() {
           log.info("这是info日志");  //
           log.warn("这是warn日志");  //警告日志
           log.error("这是error日志"); //错误日志
       }
   }  
   ```
   
5. 指定日志配置文件

   ![](https://cdn.jsdelivr.net/gh/wangd1/cdn@2.35/blogimg/logging/1566060346853.png)

- logback.xml可以被日志框架直接识别；

- logback-spring.xml无法被日志框架识别，但是可以由springboot识别，可以使用springboot的高级Profile功能。

  ```xml
  <!-- 如在logback-spring.xml中加入-->
  <springProfile name="dev">
   	 <!-- 可以指定某段配置只在某个环境下生效-->
   	 <pattern>%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX} %level [%thread] [%logger{100}.%method:%line] - %msg%n</pattern>
  </springProfile>，
  
  <!--然后修改properties配置文件的profile为dev-->
  spring.profiles.active=dev
  就可以在启动环境为dev时，日志打印的格式为对应配置的格式
  ```

所以logback[-spring].xml，格式需要自己查询。

另外logback.xml文件中配置了springProfile的话，运行会报错。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>

    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <springProfile name="dev">
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%d -- %msg%n</pattern>
            </layout>
        </springProfile>
        <springProfile name="test">
            <layout class="ch.qos.logback.classic.PatternLayout">
                <pattern>%d -----== %msg%n</pattern>
            </layout>
        </springProfile>
    </appender>

    <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!--要拦截的日志级别-->
            <level>ERROR</level>
            <!--如果匹配，则禁止-->
            <onMatch>DENY</onMatch>
            <!--如果不匹配，则允许记录-->
            <onMismatch>ACCEPT</onMismatch>
        </filter>
        <encoder>
            <pattern>%d -- %msg%n</pattern>
        </encoder>
        <!--滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径-->
            <fileNamePattern>d:/info-%d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--添加 范围 过滤-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <encoder>
            <pattern>%d -- %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>d:/error-%d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <root level="info">
        <appender-ref ref="consoleLog" />
        <appender-ref ref="fileInfoLog"/>
        <appender-ref ref="fileErrorLog"/>
    </root>
</configuration>
```

