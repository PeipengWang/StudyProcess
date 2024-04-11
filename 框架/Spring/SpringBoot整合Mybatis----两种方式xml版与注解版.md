


@[TOC](SpringBoot整合Mybatis)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、引入依赖

```c
   
        <!--jdbc-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
    
    
```
其中上面三个是必须要引入的，但是为了方便以后使用，还需要引入如下依赖：

```c
 <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.6</version>
        </dependency>
            <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.12</version>
        </dependency>
            <!--log4j-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
```



# 二、配置文件

```c
server:
  port: 8080
spring:
  datasource:
    username: root
    password: 1234
    #?serverTimezone=UTC解决时区的报错
    url: jdbc:mysql://localhost:3306/springboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true

    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
mybatis.type-aliases-package: com.uestc.wpp.pojo
mybatis.mapper-locations: classpath:mapper/*.xml
```

# 三.创建数据库对应的类




```c
package com.uestc.wpp.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private int id;
    private String name;
    private String pwd;
    private String perm;
}
```
# 四，创建持久层与对应的xml数据操作文件
## 1，持久层接口
 

```c
package com.uestc.wpp.mapper;

import com.uestc.wpp.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;

@Repository
@Mapper
public interface UserMapper {
    public User queryUserByName(String name);
}

```
# 2.持久层对应的xml文件

```c
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.uestc.wpp.mapper.UserMapper">

    <select id="queryUserByName" resultType="User" parameterType="String">
       select * from mybatis where name = #{name};
    </select>

</mapper>
```
# 五，服务层
## 1，UserService接口
```c
package com.uestc.wpp.service;

import com.uestc.wpp.pojo.User;

public interface UserService {
    public User queryUserByName(String name);
}

```
## 2，UserServiceImpl

```c
package com.uestc.wpp.service;

import com.uestc.wpp.mapper.UserMapper;
import com.uestc.wpp.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    UserMapper userMapper;
    @Override
    public User queryUserByName(String name) {
        return userMapper.queryUserByName(name);
    }
}
```
# 六，测试

```c
package com.uestc.wpp;

import com.uestc.wpp.service.UserService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class WppApplicationTests {

    @Autowired
    UserService userService;
    @Test
    void contextLoads() {
        System.out.println(userService.queryUserByName("wpp"));
    }

}

```
输出：
User(id=1, name=wpp, pwd=1234, perm=user:add)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
主要进行导入依赖，编写yml配置文件，注意要编写对应的实体类，然后编写持久层的对应xml文件中的sql语句，然后再服务层进行操作，最后得到输出
文件结构如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201117113933106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)


