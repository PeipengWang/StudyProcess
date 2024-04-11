
<font color=#999AAA >JSR-107、Spring缓存抽象、整合Redis

</font>

@[TOC](SpringBoot整合----缓存)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 一、JSR-107

Java Caching定义了5个核心接口，分别是CachingProvider, CacheManager, Cache, Entry和Expiry。
CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。
CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
Entry是一个存储在Cache中的key-value对。
Expiry每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

使用JSR107需要导入的包

```xml
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
</dependency>
```



# 二、缓存抽象？


<font color=#999AAA >Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用JCache（JSR-107）注解简化我们开发；


**几个重要概念及缓存注解**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201123105537278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)
**简要说明：**

@Cacheable注解加载方法中，那么该方法第一次会查询数据库，然后就会吧数据放在缓存中，使用Cache 进行数据的读取等操作。
@CacheEvict删除缓存，例如根据id删除用户，那么也要删除缓存中的用户信息
@CachePut更新缓存，例如更新用户信息后，同时也要更新缓存中的用户信息



# 三、环境搭建
## 1.创建数据库

```sql
SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for department
-- ----------------------------
DROP TABLE IF EXISTS `department`;
CREATE TABLE `department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `departmentName` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for employee
-- ----------------------------
DROP TABLE IF EXISTS `employee`;
CREATE TABLE `employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `lastName` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `gender` int(2) DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201123111912122.png#pic_center)

## 2.导入依赖

```xml
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
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.6</version>
        </dependency>
```

## 3.配置文件

```yml
server:
  port: 8080
spring:
  datasource:
    username: root
    password: 1234
    #?serverTimezone=UTC解决时区的报错
    url: jdbc:mysql://localhost:3306/spring_cache?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver


```
##  4. 整合Mybatis编写bean，mapper，service，controller
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201123150834884.png#pic_center)
运行结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201123150948323.png#pic_center)
## 5. 开启基于注解的缓存

```java
package com.uestc.wpp;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@MapperScan("com.uestc.wpp.mapper")
@SpringBootApplication
@EnableCaching//开启这个
public class WppApplication {

    public static void main(String[] args) {
        SpringApplication.run(WppApplication.class, args);
    }

}

```

## 6. 标注注解缓存

不加缓存前

```xml
进入控制层查找一个employee
进入服务层查找
2020-11-23 15:23:41.013 DEBUG 4612 --- [nio-8080-exec-2] c.u.w.mapper.EmployeeMapper.getEmpById   : ==>  Preparing: select * from employee where id = ? 
2020-11-23 15:23:41.013 DEBUG 4612 --- [nio-8080-exec-2] c.u.w.mapper.EmployeeMapper.getEmpById   : ==> Parameters: 1(Integer)
2020-11-23 15:23:41.018 DEBUG 4612 --- [nio-8080-exec-2] c.u.w.mapper.EmployeeMapper.getEmpById   : <==      Total: 1
```
加上缓存

```java
package com.uestc.wpp.service;

import com.uestc.wpp.bean.Employee;
import com.uestc.wpp.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    @Cacheable
    public Employee getEmpById(Integer id){
        System.out.println("进入服务层查找");
        return employeeMapper.getEmpById(id);
    }
}

```


加入缓存后连续两次查询的结果：

```c
进入控制层查找一个employee
进入服务层查找
2020-11-23 15:35:57.047  INFO 6260 --- [nio-8080-exec-1] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2020-11-23 15:35:57.416  INFO 6260 --- [nio-8080-exec-1] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2020-11-23 15:35:57.426 DEBUG 6260 --- [nio-8080-exec-1] c.u.w.mapper.EmployeeMapper.getEmpById   : ==>  Preparing: select * from employee where id = ? 
2020-11-23 15:35:57.452 DEBUG 6260 --- [nio-8080-exec-1] c.u.w.mapper.EmployeeMapper.getEmpById   : ==> Parameters: 1(Integer)
2020-11-23 15:35:57.481 DEBUG 6260 --- [nio-8080-exec-1] c.u.w.mapper.EmployeeMapper.getEmpById   : <==      Total: 1
进入控制层查找一个employee
进入控制层查找一个employee
```
这时候会发现不会再次进入服务层执行SQL语句了，可以直接从缓存里拿数据。

# 四、缓存的工作原理
## 1. 自动配置类CacheAutoConfiguration

```c
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureAfter({ CouchbaseDataAutoConfiguration.class, HazelcastAutoConfiguration.class,
		HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class })
@Import({ CacheConfigurationImportSelector.class, CacheManagerEntityManagerFactoryDependsOnPostProcessor.class })
public class CacheAutoConfiguration {
}
```
其中`@Import({ CacheConfigurationImportSelector.class, CacheManagerEntityManagerFactoryDependsOnPostProcessor.class })`
## 2. 运行流程
1，在方法运行之前，先去查询Cache(缓存组件)，按照cacheNames指定的名字获取；
（CacheManager先获取相应的缓存），第一次取缓存如果没有Cache会自动创建；
2，去Cache中查找缓存的内容，使用一个key，默认方法的参数，可以是按照某种策略进行，默认是使用keyGenerator策略进行，默认使用SimpleKeyGenerator生成key。
SimplekeyGenerator生成key：
如果没有参数：key=new SimpleKey();
如果有一个参数：key=参数值
如果有多个参数：key=new SimpleKey（params）；
3，没有查到缓存就调用目标方法。
4，将目标方法返回结果，放进缓存。

核心：
1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurreneMapCache】组件
2）、key使用keyGengeator生成的，默认是SimpleGenerator

# 五、 Cacheable的属性
1）. cacheNames
```java
	@AliasFor("cacheNames")
	String[] value() default {};
```
指定缓存组件的名字
2）. key
```java
String key() default "";
```
指定缓存的数据，可以用它来指定，如果不指定。默认是使用该参数的值
key可以取得值
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201123152859882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)
3）. condition
```java
String condition() default "";
```
缓存条件，可以为空，返回true或者false，只有满足条件才会进行缓存或者清除缓存，在调用方法前都能判断
eg：`@Cacheable（condition="#id>0"）`只有id属性大于0的数据才会进行缓存
4）. unless
```java
String unless() default "";
```
否决缓存条件，跟condition相反，只有条件是false才会进行缓存。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201123152809835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)
# 六、@CachePut与@CacheEvict

@CachePut：既调用方法，又更新缓存数据，修改了数据库的某个数据，同时更新缓存
先调用目标方法，然后将目标方法的结果放入缓存

@CacheEvict：缓存清除，在删除数据后会将缓存中的记录也删除掉。

```java
package com.uestc.wpp.service;

import com.uestc.wpp.bean.Employee;
import com.uestc.wpp.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;
@CacheConfig(cacheNames = "emp")
@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    @Cacheable( key = "#id")
    public Employee getEmpById(Integer id){
        System.out.println("进入服务层查找");
        return employeeMapper.getEmpById(id);
    }
    @CachePut(key = "#result.id")
    public Employee updateEmp(Employee employee){
        employeeMapper.updateEmp(employee);
        System.out.println("服务层更新完成");
        return employeeMapper.getEmpById(employee.getId());
    }
    @CacheEvict( key="#id")
    public void deleteEmp(Integer id){
        employeeMapper.deleteEmpById(id);
    }
}
```
# 七、 搭建Redis环境
## 1. 安装Redis
[Redis命令大全](http://www.redis.cn/commands.html)
[安装和常用命令](https://blog.csdn.net/Artisan_w/article/details/108337536)

## 2. 引入依赖

```xml
        <!--redis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```
## 3. 配置Redis

```xml
spring.redis.host=服务器地址
spring.redis.port=6379
```
## 4. 测试程序

```java
    @Autowired
    StringRedisTemplate stringRedisTemplate; //操作字符串

    @Autowired
    RedisTemplate redisTemplate;//操作k-v对象

    @Test
    void test02() {
        Employee employee = employeeMapper.getEmpById(1);
        System.out.println(employee);
        redisTemplate.opsForValue().set("man1",employee);
        System.out.println(redisTemplate.opsForValue().get("man1"));
    }
```
出现错误如下：

```xml
nested exception is java.lang.IllegalArgumentException: DefaultSerializer requires a Serializable payload but received an object of type [com.uestc.wpp.bean.Employee]
```
需要将类序列化

```java
public class Employee implements Serializable 
```
再次测试得到输出结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112415332266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124153517923.png#pic_center)
在上面的redis可视化可以看到，需要用json工具转化一下：序列化
## 5.json 序列化
重新编写序列化工具类

```java
package com.uestc.wpp.config;

import com.uestc.wpp.bean.Employee;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

@Configuration
public class MyRedisConfig {
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Employee> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> emr = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(emr);
        return template;
    }

}

```
利用我们定义的工具类进行序列化

```java
@Autowired
RedisTemplate<Object,Employee> redisTemplate1;
    @Test
    void test05() {
        Employee employee = employeeMapper.getEmpById(1);
        System.out.println(employee);
        redisTemplate1.opsForValue().set("man2",employee);
        System.out.println(redisTemplate1.opsForValue().get("man2"));
    }
```

可视化输出redis结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124160005833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)
# 八、测试缓存
## 1. 原理
CacheManager：Cache缓存组件来实际给缓存中存储数据
默认使用的是SimpleCacheManager，但是如果我们引入Redis的starter之后就会变为RedisCacheManager，RedisCacheManager帮我们创建RedisCache来作为缓存组件，RedisCache通过操作redis缓存数据
因此我们可以直接运行主程序来测试：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124162227497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)
从上述可以看出，默认保存的数据k-v都是object；利用序列化来保存的，即默认创建RedisCacheManager操作redis的时候使用的是RedisTemplate<Object，Object>
源代码如下：
```java
	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}
```
默认使用的是jdk的序列化机制
## 2. 自定义CacheManager
自定义序列化配置类：
```java
package com.uestc.wpp.config;

import com.uestc.wpp.bean.Employee;

import org.springframework.cache.CacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
public class MyRedisConfig {
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Employee> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> emr = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(emr);
        return template;
    }
@Bean
    public CacheManager employeeCacheManager(RedisConnectionFactory redisConnectionFactory){
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofDays(1))
                .disableCachingNullValues()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));


        return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(config).build();
    }

}

```
输出结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124165558284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)

## 3. 需要多个CacheManager时
在2.0之前需要定义多个Manager，2.0后好像不需要了