


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 一、日志框架？
市面上的日志框架：
JUL、JCL、jboss-logging、Logback、Log4j、Log4j2、SLF4j....

|日志门面（抽象层）|日志实现  |
|--|--|
|  JCL、SLF4j、jboss-logging|Log4j、JUL、Log4j2、Logback  |
左边选择一个抽象层，右边选择一个实现；
JCL比较落后，boss-logging天生不是给开发者用的。
JUL是java自带的，Logback是Log4j的升级版，Log4j2是apcche公司开发的借了Log4j之名。
日志门面：SLF4j；
日志实现：Logback；

SpringBoot：底层是Spring框架，Spring默认使用JCL；
SpringBoot选用SLF4j和Logback

# 二、SLF4j的使用
## 1.如何在系统中使用SLF4j
以后开发的时候，日志记录方法的调用，不应该直接调用日志实现类，而是调用日志抽象层里面的方法。
[SLF4j官方网站http://www.slf4j.org/](http://www.slf4j.org/)

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
由代码可以看出，要给系统导入slf4j的jar和logback的实现jar。
每个日志框架都有自己的配置文件，在使用slf4j以后配置文件还是做成日志实现框架自己本身的配置文件。

## 2.遗留问题
不同的框架有不同的日志框架：
a（slf4j+logback），Spring（common-logging）、Hibernate（jboss-logging）、Mybatis
统一日志框架，即使别的框架统一使用slf4j作为日志输出。
<font color=red >
（1），将系统中其他日志框架先排除出去
（2），用中间包替换原有的日志框架
（3），我们导入slf4j其他的实现
# 三.SpringBoot的日志关系
SpringBoot自带的日志框架

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-logging</artifactId>
      <version>2.4.0</version>
      <scope>compile</scope>
    </dependency>
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/202011201515171.png)

1）、SpringBoot底层也是使用slf4j+logback的方式进行日志记录
2）、SpringBoot把其他日志都转换成了slf4j
3）、中间替换包

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201120151958842.png)
4)、如果我们要引入其他框架，一定要把这个框架的默认日志移除掉。

# 四. 日志的使用
## 1. 默认配置
SpringBoot默认帮助我们配置好了日志
我们可以直接构造记录器来进行使用

```java
package com.uestc.springboot;

import org.junit.jupiter.api.Test;

import org.slf4j.Logger;//注意引入的为slf4j而不是jdk自带的
import org.slf4j.LoggerFactory;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class SpringbootLogging01ApplicationTests {

   //记录器
    Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    void contextLoads() {
        logger.trace("这是一个trace级别的日志。。。");
        logger.info("这是一个info级别的日志。。。");
        logger.debug("这是一个debug级别的日志。。。");
        logger.warn("这是一个warn级别的日志。。。");
        logger.error("这是一个error级别的日志。。。");

    }

}

```
输出为：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201120154807936.png)
这时会发现只显示了debug，warn和error三个级别的日志，因此需要注意以下几点：
（1）日志的级别为：trace<info<debug<warn<error
（2）SpringBoot默认是denug级别的输出，因此不会显示info与trace级别的日志
（3)可以在配置文件中调整日志输出级别

```properties
logging.level.com.uestc.springboot=trace
```
这样就可以把日志级别调整为trace级别
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201120155305820.png)

## 2，日志配置位置
```properties
logging.file.name=springboot.log
logging.file.path=/home/ap/ywzt/log
```
分别用上面两个配置设置日志文件的名字与位置

## 3. 其他
```properties
#控制台日志输出格式
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger : %msg%n
# 修改输出到文件的日志格式
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger : %msg%n
# 日志文件大小
logging.file.max-size=10MB
#保留的日志时间
logging.file.max-history=10
```

日志输出格式：
%d{HH: mm:ss.SSS}——日志输出时间。
%thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用。
%-5level——日志级别，并且使用5个字符靠左对齐。
%logger{36}——日志输出者的名字。
%msg——日志消息。
%n——平台的换行符。


## 4. 自定义日志配置
使用不同的日志框架要建立不同的配置文件，如下所示
|Logging System|Customization  |
|--|--|
|Logback  |logback.xml ， logback-spring.xml， logback-spring.groovy， logback.groovy  |
|Log4j2|log4j2.xml，log4j2-spring.xml|
|JDK|logging.properties|

logback.xml:直接就被日志框架识别了
logback-spring.xml：日志框架就不加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能。

## 5. 切换日志框架
可以按照slf4j的日志适配图，进行相关切换：
slf4j+log4j的方式







