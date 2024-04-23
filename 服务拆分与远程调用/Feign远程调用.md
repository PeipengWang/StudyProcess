# Feign远程调用
Fegin的使用步骤如下：

1）引入依赖
我们在order-service服务的pom文件中引入feign的依赖：
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
2）引入注解
```
@EnableFeignClients
@MapperScan("cn.itcast.order.mapper")
@SpringBootApplication
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
```
3）编写Feign的客户端
在order-service中新建一个接口，内容如下：
```
package cn.itcast.order.client;

import cn.itcast.order.pojo.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```
这个客户端主要是基于SpringMVC的注解来声明远程调用的信息，比如：

服务名称：userservice
请求方式：GET
请求路径：/user/{id}
请求参数：Long id
返回值类型：User
这样，Feign就可以帮助我们发送http请求，无需自己使用RestTemplate来发送了。
4）测试：
```
package cn.itcast.order.service;

import cn.itcast.order.client.UserClient;
import cn.itcast.order.mapper.OrderMapper;
import cn.itcast.order.pojo.Order;
import cn.itcast.order.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;

    @Autowired
    private UserClient userClient;
    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        // 2.远程查询user
        User user = userClient.findById(order.getUserId());
        // 3 封装user到order
        order.setUser(user);
        // 4.返回
        return order;
    }
}

```
5）总结
使用Feign的步骤：
① 引入依赖
② 添加@EnableFeignClients注解
③ 编写FeignClient接口
④ 使用FeignClient中定义的方法代替RestTemplate
# 自定义配置
![](https://raw.githubusercontent.com/PeipengWang/picture/master/20211225225036282_12232.png)
1)在 application.yml 文件中进行配置，包括全局配置和针对某个服务的配置，如下所示
①全局配置
```
feign:  
  client:
    config: 
      default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```
②局部配置
```
feign:  
  client:
    config: 
      userservice: # 针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```
而日志的级别分为四种：
NONE：不记录任何日志信息，这是默认值。
BASIC：仅记录请求的方法，URL以及响应状态码和执行时间
HEADERS：在BASIC的基础上，额外记录了请求和响应的头信息
FULL：记录所有请求和响应的明细，包括头信息、请求体、元数据。
2）Java代码方式
也可以基于Java代码来修改日志级别，先声明一个类，然后声明一个Logger.Level的对象：
```
public class DefaultFeignConfiguration  {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC; // 日志级别为BASIC
    }
}
```
如果要全局生效，将其放到启动类的@EnableFeignClients这个注解中：
`@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration .class) `
如果是局部生效，则把它放到对应的@FeignClient这个注解中：
`
@FeignClient(value = "userservice", configuration = DefaultFeignConfiguration .class) `

# Feign最佳实践
所谓最近实践，就是使用过程中总结的经验，最好的一种使用方式。
1）继承方式
一样的代码可以通过继承来共享：
定义一个API接口，利用定义方法，并基于SpringMVC注解做声明。
Feign客户端和Controller都集成改接口
优点：
简单
实现了代码共享
缺点：
服务提供方、服务消费方紧耦合
参数列表中的注解映射并不会继承，因此Controller中必须再次声明方法、参数列表、注解
2）抽取方式
将Feign的Client抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用。
例如，将UserClient、User、Feign的默认配置都抽取到一个feign-api包中，所有微服务引用该依赖包，即可直接使用。
①抽取
首先创建一个module，命名为feign-api：
②在feign-api中然后引入feign的starter依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
然后，order-service中编写的UserClient、User、DefaultFeignConfiguration都复制到feign-api项目中
③在order-service中使用feign-api
首先，删除order-service中的UserClient、User、DefaultFeignConfiguration等类或接口。
在order-service的pom文件中中引入feign-api的依赖：
```
<dependency>
    <groupId>cn.itcast.demo</groupId>
    <artifactId>feign-api</artifactId>
    <version>1.0</version>
</dependency>
```
修改order-service中的所有与上述三个组件有关的导包部分，改成导入feign-api中的包
④解决扫描包问题
方式一：
指定Feign应该扫描的包：
```
@EnableFeignClients(basePackages = "cn.itcast.feign.clients")
```
方式二：
指定需要加载的Client接口：
```
@EnableFeignClients(clients = {UserClient.class})
```