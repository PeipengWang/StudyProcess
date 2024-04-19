


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 概述
1，大多数应用中，可以通过消息服务中间件来提升系统异步通信、扩展解耦能力
2，消息服务的两个重要概念：
消息代理（message broker）和目的地（destination）当消息发送者发送消息后，将由消息代理接管，消息代理保证消息传递到指定目的地。
3，消息队列主要有两种形式的目的地
队列：点对点消息通信
主题：发布（publish）/订阅（subscribe）通信消息；
4，点对点式
消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息被读取后移出队列。
5，发布订阅式
发送者发送消息到主题，多个接受者（订阅者）监听（订阅）这个主题，那么就会在消息队列就会在消息到达时同时接收消息。
6，JMS（Java Meeage Service）java消息服务
基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现。
7，AMQP（Advanced Message Queuing Protocol）
高级消息队列协议，也是一个消息代理的规范，兼容JMS；
RabbitMQ是AMQP的实现。
8. Spring支持
– spring-jms提供了对JMS的支持
– spring-rabbit提供了对AMQP的支持
– 需要ConnectionFactory的实现来连接消息代理
– 提供JmsTemplate、RabbitTemplate来发送消息
– @JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发
布的消息
– @EnableJms、@EnableRabbit开启支持
9. Spring Boot自动配置
– JmsAutoConfiguration
– RabbitAutoConfiguration
# 一、RabbitMQ
## 1，RabbitMQ简介：
RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。
## 2，核心概念
**Message**
消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组
成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出
该消息可能需要持久性存储）等。
**Publisher**
**消息的生产者**，也是一个向交换器发布消息的客户端应用程序。
**Exchange（决定消息发到哪）**
交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有
所区别
**Queue**
消息队列，用来保存消息直到发送给消费者。它是**消息的容器**，也是消息的终点。一个消息
可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
**Binding**
绑定，用于**消息队列和交换器之间的关联**。一个绑定就是基于路由键将交换器和消息队列连
接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
Exchange 和Queue的绑定可以是多对多的关系。
**Connection**
网络连接，比如一个TCP连接。
**Channel(解决多路复用)**
信道，**多路复用连接中的一条独立的双向数据流通道**。信道是建立在真实的TCP连接内的虚
拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这
些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所
以引入了信道的概念，以复用一条 TCP 连接。
**Consumer**
消息的消费者，表示一个从消息队列中取得消息的**客户端应用程序**。
**Virtual Host**
虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加
密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有
自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，
**RabbitMQ 默认的 vhost 是 /** 。
**Broker（消息代理）**
表示消息队列服务器实体
整体通信连接方式如下图所示：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2020112511255768.png)



# 二、RabbitMQ运行机制
## 1. AMQP 中的消息路由
• AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125112804235.png)

## 2.Exchange 类型
Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers 。headers 匹配AMQP 消息的 header而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：
### （1）Direct Exchange（点对点）
消息中的路由键（routing key）如果和 Binding 中的 bindingkey 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会发“dog.puppy”，也不会转发“dog.guard”等等。**它是完全匹配、单播的模式。**
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125144953299.png)

### （2）Fanout Exchange（广播）
每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。**很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发息是最快的。**
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125145025286.png)


### （3）Topic Exchange（模糊匹配）
topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些 单词之间用点隔开。它同样也会识别两个通配符：**符号“#”和符号“* ”。 # 匹配 0 个或多个单词， *匹配一个单词**
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125145121123.png)


# 三、RabbitMQ的安装配置
## 1. 安装
利用docker下载：[docker基本用法](https://blog.csdn.net/Artisan_w/article/details/109904812)

```shell
docker pull rabbitmq:3-management
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125202440828.png)
查看已下载的镜像

```shell
docker images
```
运行

```shell
docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq 87505dc99f21
```

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125202234214.png)
外部访问：服务器地址：15672
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125203116776.png)

登陆：
Username：guest
password：guest
登陆界面
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125203250213.png)

## 2.配置
### （1）配置Exchange
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125203913512.png)
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125204108251.png)

### （2）配置queue
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125204441305.png)
### （3）Exchanger绑定queue
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125205237937.png)
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125205723320.png)

### （4）测试发送
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125210127128.png)
消息队列
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125210207814.png)
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201125210318559.png)

# 四、 SpringBoot整合RabbitMQ
## 1. 导入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
        </dependency>
```
##  2. 配置文件
application.properties
```properties
spring.rabbitmq.host=服务器地址
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
#spring.rabbitmq.virtual-host="/"

```
## 3. 程序测试

```java
package com.uestc.wpp;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.HashMap;
import java.util.Map;

@SpringBootTest
class WppApplicationTests {

    @Autowired
    RabbitTemplate rabbitTemplate;
//    点对点（单播）
    @Test
    void test01() {
       //Message需要自己构造一个；定义消息体内容和消息头
       // rabbitTemplate.send(exchange,routeKey,message);
        //object默认为消息体
        //rabbitTemplate.convertAndSend(exchange,routKey,object);
        rabbitTemplate.convertAndSend("exchange.direct","wpp","hello springboot");
        Map<String,String> map = new HashMap<>();
        map.put("msg1","第一条消息");
        map.put("msg2","第二条消息");
        rabbitTemplate.convertAndSend("exchange.direct","wpp.news",map);

    }

}

```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2020112620304630.png)
## 4.  json序列化
编写一个配置文件将序列化方式配置为json方式
```java
package com.uestc.wpp.config;

import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyAMQPConfig {
    @Bean
    public MessageConverter messageConverter(){//注意导入的包
        return new Jackson2JsonMessageConverter();
    }
}

```
通过可视化界面验证可以看出：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201126204927468.png)
广播的发送方式与单播的代码类似，但是广播无论法哪个队列，所有的exchange有绑定的queue都会收到。

## 5.  @RabbitListenner与@EnableRabbit
 @RabbitListenner：用来打开信道监听


```java
package com.uestc.wpp;

import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@EnableRabbit //开启基于注解的RabbitMQ
@SpringBootApplication
public class WppApplication {

    public static void main(String[] args) {
        SpringApplication.run(WppApplication.class, args);
    }

}

```

@EnableRabbit：用来作为监听某个队列的对象来用的，例如，定义一个Book类，如下所示：

```java
package com.uestc.wpp.bean;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Book {
    private String bookName;
    private String authorName;
}

```
发送一个Book对象后，监听这个wpp.news这个queue的book对象消息

```java
package com.uestc.wpp.service;

import com.uestc.wpp.bean.Book;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;



@Service
public class BookService {
    @RabbitListener(queues = "wpp.news")
    public void receive(Book book){
        System.out.println("收到消息"+book);
    }
}

```
启动主程序，结果如下所示。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201126211903162.png)

## 6. AmdqAdmin
AmdqAdmin用来创建和删除Queue，Exchange和Binding，可以用代码方式来完成我们上述在可视化界面进行的操作。
代码：
### （1）. 创建Exchange
```java
@Autowired
    AmqpAdmin amqpAdmin;

    @Test
    public void createExchange(){
        amqpAdmin.declareExchange(new DirectExchange("admin.exchange"));
    }
```

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201126215346700.png)
### （2）. 创建queue
代码：

```java
   @Autowired
    AmqpAdmin amqpAdmin;
        @Test
    public void createQueue(){
        amqpAdmin.declareQueue(new Queue("admin.queue"));
    }
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201126215619336.png)
### （3）. 绑定

```java
  @Test
    public void createBinding(){
        amqpAdmin.declareBinding(new Binding("admin.queue",Binding.DestinationType.QUEUE,"admin.exchange","amqp.aaa",null));
    }
  
```
其中Binding的参数代码如下：

```java
public Binding(String destination, DestinationType destinationType, String exchange, String routingKey,
			@Nullable Map<String, Object> arguments) {
```
分别为目的地，目的地类型，交换器名称，routingKey名字和arguments。
绑定结果演示：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201126220244738.png)

