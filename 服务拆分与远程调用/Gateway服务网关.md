# Gateway服务网关
Gateway网关是我们服务的守门神，所有微服务的统一入口。
网关的核心功能特性：
身份认证和权限校验
服务路由和负载均衡
请求限流
SpringCloud实现方式：gateway，zuul
zuul时基于servlet实现的，是一种阻塞式编程，而gateway是基于Spring5提供的WebFlux，属于响应式编程的实现，具有更好的性能。

## 搭建网关服务
1、 创建SpringBoot工程gateway，引入网关依赖
```
<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos服务发现依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```
2、 编写启动类
```
package cn.itcast.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(GatewayApplication.class, args);
	}
}
```
3、 编写基础配置和路由规则
创建application.yml文件，内容如下：
```
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: user-service # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
```
我们将符合Path 规则的一切请求，都代理到 uri参数指定的地址。

本例中，我们将 /user/**开头的请求，代理到lb://userservice，lb是负载均衡，根据服务名拉取服务列表，实现负载均衡。
4、 启动网关服务进行测试

5、流程总结
![](https://raw.githubusercontent.com/PeipengWang/picture/master/20211226175412204_21744.png)
总结：
网关搭建步骤：
创建项目，引入nacos服务发现和gateway依赖
配置application.yml，包括服务基本信息、nacos地址、路由
路由配置包括：
路由id：路由的唯一标示
路由目标（uri）：路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
路由断言（predicates）：判断路由的规则，
路由过滤器（filters)：对请求或响应做处理
接下来，就重点来学习路由断言和路由过滤器的详细知识
## 断言工厂
我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断的条件
例如Path=/user/**是按照路径匹配，这个规则是由
org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory类来
处理的，像这样的断言工厂在SpringCloudGateway还有十几个:
![](_v_images/20211226184024146_822.png =1062x)

## 过滤器工厂（目前32个）
GatewayFilter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理：
![](_v_images/20211226183930950_27577.png =1016x)
下面我们以AddRequestHeader 为例来讲解。
### 服务过滤器
只需要修改gateway服务的application.yml文件，添加路由过滤即可：
```
spring:
  cloud:
    gateway:
      routes:
      - id: user-service 
        uri: lb://userservice 
        predicates: 
        - Path=/user/** 
        filters: # 过滤器
        - AddRequestHeader=Truth, testRequestHeader # 添加请求头
```
测试：
修改UserController.java，添加获取请求头数据：
```
    @GetMapping("/{id}")
    public User queryById(@PathVariable("id") Long id,
                          @RequestHeader("Truth") String Truth) {
        System.out.println("Request Headers:"+Truth);
        return userService.queryById(id);
    }
```
获得的数据如下：
![](_v_images/20211226185509423_2190.png =1333x)
### 默认过滤器
如果要对所有的路由都生效，则可以将过滤器工厂写到default下。格式如下：
```
spring:
  cloud:
    gateway:
      routes:
      - id: user-service 
        uri: lb://userservice 
        predicates: 
        - Path=/user/**
      default-filters: # 默认过滤项
      - AddRequestHeader=Truth, Itcast is freaking awesome! 
```
## 全局过滤器
网关提供了32种，但每一种过滤器的作用都是固定的。如果我们希望拦截请求，做自己的业务逻辑则没办法实现。
全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。区别在于GatewayFilter通过配置定义，处理逻辑是固定的；而GlobalFilter的逻辑需要自己写代码实现。
定义方式是实现GlobalFilter接口。
```
public interface GlobalFilter {
    /**
     *  处理当前请求，有必要的话通过{@link GatewayFilterChain}将请求交给下一个过滤器处理
     *
     * @param exchange 请求上下文，里面可以获取Request、Response等信息
     * @param chain 用来把请求委托给下一个过滤器 
     * @return {@code Mono<Void>} 返回标示当前过滤器业务结束
     */
    Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
}
```
在filter中编写自定义逻辑，可以实现下列功能：
登录状态判断
权限校验
请求限流等
### 自定义全局过滤器
需求：定义全局过滤器，拦截请求，判断请求的参数是否满足下面条件：
参数中是否有authorization，
authorization参数值是否为admin
如果同时满足则放行，否则拦截
实现：
在gateway中定义一个过滤器：
```
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Order(-1)
@Component
public class AuthorizeFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1.获取请求参数
        MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();
        // 2.获取authorization参数
        String auth = params.getFirst("authorization");
        // 3.校验
        if ("admin".equals(auth)) {
            // 放行
            return chain.filter(exchange);
        }
        // 4.拦截
        // 4.1.禁止访问，设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
        // 4.2.结束处理
        return exchange.getResponse().setComplete();
    }
}
```
各种过滤器执行顺序：
![](_v_images/20211226214002085_9737.png =988x)
排序的规则是什么呢？
每一个过滤器都必须指定一个int类型的order值，order值越小，优先级越高，执行顺序越靠前。
GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。
### 跨域问题
跨域：域名不一致就是跨域，主要包括：
域名不同： www.taobao.com 和 www.taobao.org 和 www.jd.com 和 miaosha.jd.com
域名相同，端口不同：localhost:8080和localhost8081
跨域问题：浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题
解决方案：CORS，这个以前应该学习过，这里不再赘述了。不知道的小伙伴可以查看https://www.ruanyifeng.com/blog/2016/04/cors.html
在gateway服务的application.yml文件中，添加下面的配置：
```
spring:
  cloud:
    gateway:
      # 。。。
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins: # 允许哪些网站的跨域请求 
              - "http://localhost:8090"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```

