# 1. RabbitMQ
## 1.1. 同步通讯与异步通讯
同步通讯：实时响应，例如视频电话  
异步通讯：不需要马上回复，例如收发短信  
同步通讯时效性较强，可以立即得到结果，但是也会有一定问题，如下所示为一个支付服务与其调用的服务，在此项服务中用户发送支付请求，支付服务会依次远程调用订单服务，仓储服务和短息服务，在这个调用过程是一个链式的过程，总用时为500ms。  
在这个服务中，每次修改其它服务，就要相应的去修改支付服务的代码，会产生耦合度高的问题；在调用过程中，调用者为了进行实时响应，会一直处在等待状态，等到调用的服务返回数据，会产生性能和吞吐能力下降的问题；同时，调用链的每个服务都要等待其它服务调用完成，在高并发条件下产生额外的资源的浪费；最后，调用链的一个服务失败会导致整个级联的失败，如同多米诺骨牌一样迅速崩塌，导致整个微服务失败。由此产生了异步通讯的解决方案  
<img src="https://raw.githubusercontent.com/PeipengWang/picture/master/20220103211646396_843.png" alt="" width="946">

总结：  
同步调用的优点：  
时效性较强，可以立即得到结果  

同步调用的问题：  
耦合度高  
性能和吞吐能力下降  
有额外的资源消耗  
有级联失败问题  

## 1.2. 异步通讯的优缺点  
异步调用则可以避免上述问题  
我们以购买商品为例，用户支付后需要调用订单服务完成订单状态修改，调用物流服务，从仓库分配响应的库存并准备发货。  
在事件模式中，支付服务是事件发布者（publisher），在支付完成后只需要发布一个支付成功的事件（event），事件中带上订单id。  
订单服务和物流服务是事件订阅者（Consumer），订阅支付成功的事件，监听到事件后完成自己业务即可。  
为了解除事件发布者与订阅者之间的耦合，两者并不是直接通信，而是有一个中间人（Broker）。发布者发布事件到Broker，不关心谁来订阅事件。订阅者从Broker订阅事件，不关心谁发来的消息。  
<img src="https://raw.githubusercontent.com/PeipengWang/picture/master/20220103214031901_4452.png" alt="" width="781">

Broker 是一个像数据总线一样的东西，所有的服务要接收数据和发送数据都发到这个总线上，这个总线就像协议一样，让服务间的通讯变得标准和可控。  
好处：  
吞吐量提升：无需等待订阅者处理完成，响应更快速  
故障隔离：服务没有直接调用，不存在级联失败问题  
调用间没有阻塞，不会造成无效的资源占用  
耦合度极低，每个服务都可以灵活插拔，可替换  
流量削峰：不管发布事件的流量波动多大，都由Broker接收，订阅者可以按照自己的速度去处理事件  
缺点：  
架构复杂了，业务没有明显的流程线，不好管理    
需要依赖于Broker的可靠、安全、性能    
好在现在开源软件或云平台上 Broker 的软件是非常成熟的，比较常见的一种就是我们今天要学习的MQ技术。    
## 1.3. 几种MQ的对比  
<img src="https://raw.githubusercontent.com/PeipengWang/picture/master/20220103215057625_18647.png" alt="" width="919">

## 1.4. docker安装运行RabbitMQ 流程
1、安装docker  
2、安装RabbitMQ  
方式一：在线拉取  
```
docker pull rabbitmq:3-management
```
方式二：从本地加载  
上传到虚拟机中后，使用命令加载镜像即可：  
```
docker load -i mq.tar
```
查看是否创建镜像成功  
```
dockers images
```
3、运行RabbitMQ  
如果已经存在则执行docker start dockerid  
不存在则创建一个： 
```
 docker run \
 -e RABBITMQ_DEFAULT_USER=itcast \
 -e RABBITMQ_DEFAULT_PASS=123321 \
 --name wmq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```
注意要放开防火墙  
查看是否运行成功   
```
docker ps
```
4、进入RabbitMQ目录  
```
docker exec -it 991793c8abeb /bin/bash
```
5、使得网站可以访问  
```
rabbitmq-plugins enable rabbitmq_management
```
6、访问  
ip:15672  
## 1.5. RabbitMQ的几个概念  
channel：操作MQ的工具  
exchange：路由消息到队列中  
queue：缓存消息  
virtual host： 虚拟主机，对queue、exchange等资源的逻辑分组  
## 1.6. 五种模型  
### 1.6.1. 基本消息队列  
![](https://raw.githubusercontent.com/PeipengWang/picture/master/20220209215426016_2296.png)
publisher：消息发布者，将消息发布到队列  
queue：接收消息并缓存  
consumer：订阅队列，处理队列中的消息  

## 1.7. 基本使用  
```
package cn.itcast.mq.helloworld;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import org.junit.Test;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class PublisherTest {
    @Test
    public void testSendMessage() throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("101.42.93.208");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("itcast");
        factory.setPassword("123321");
        // 1.2.建立连接
        Connection connection = factory.newConnection();

        // 2.创建通道Channel
        Channel channel = connection.createChannel();

        // 3.创建队列
        String queueName = "simple.queue";
        channel.queueDeclare(queueName, false, false, false, null);

        // 4.发送消息
        String message = "hello, rabbitmq!";
        channel.basicPublish("", queueName, null, message.getBytes());
        System.out.println("发送消息成功：【" + message + "】");

        // 5.关闭通道和连接
        channel.close();
        connection.close();

    }
}

```

# 2. Spring AMQP
## 2.1. 简单入门
1、引入依赖  
在父工程mq-demo中引入依赖  
```
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
### 2.1.1. 消息发送  
首先配置MQ地址，在publisher服务的application.yml中添加配置：    
```
spring:
  rabbitmq:
    host: 192.168.150.101 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: itcast # 用户名
    password: 123321 # 密码
```
然后在publisher服务中编写测试类SpringAmqpTest，并利用RabbitTemplate实现消息发送：  
```
package cn.itcast.mq.spring;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringAmqpTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testSimpleQueue() {
        // 队列名称
        String queueName = "simple.queue";
        // 消息
        String message = "hello, spring amqp!";
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message);
    }
}
```
### 2.1.2. 接受消息
首先配置MQ地址，在consumer服务的application.yml中添加配置：
```
spring:
  rabbitmq:
    host: 192.168.150.101 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: itcast # 用户名
    password: 123321 # 密码
```
然后在consumer服务的cn.itcast.mq.listener包中新建一个类SpringRabbitListener，代码如下：
```
package cn.itcast.mq.listener;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class SpringRabbitListener {

    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueueMessage(String msg) throws InterruptedException {
        System.out.println("spring 消费者接收到消息：【" + msg + "】");
    }
}
```
## 2.2. WorkQueue消息队列   
Work queues，也被称为（Task queues），任务模型。简单来说就是让多个消费者绑定到一个队列，共同消费队列中的消息。  
当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。  
此时就可以使用work 模型，多个消费者共同处理消息处理，速度就能大大提高了。  
### 2.2.1. 消息发送  
这次我们循环发送，模拟大量消息堆积现象。  

在publisher服务中的SpringAmqpTest类中添加一个测试方法：  
```
/**
     * workQueue
     * 向队列中不停发送消息，模拟消息堆积。
     */
@Test
public void testWorkQueue() throws InterruptedException {
    // 队列名称
    String queueName = "simple.queue";
    // 消息
    String message = "hello, message_";
    for (int i = 0; i < 50; i++) {
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message + i);
        Thread.sleep(20);
    }
}
```
### 2.2.2. 消息接收
要模拟多个消费者绑定同一个队列，我们在consumer服务的SpringRabbitListener中添加2个新的方法：  
```
@RabbitListener(queues = "simple.queue")
public void listenWorkQueue1(String msg) throws InterruptedException {
    System.out.println("消费者1接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(20);
}

@RabbitListener(queues = "simple.queue")
public void listenWorkQueue2(String msg) throws InterruptedException {
    System.err.println("消费者2........接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(200);
}
```
注意到这个消费者sleep了1000秒，模拟任务耗时。
启动ConsumerApplication后，在执行publisher服务中刚刚编写的发送测试方法testWorkQueue。
可以看到消费者1很快完成了自己的25条消息。消费者2却在缓慢的处理自己的25条消息。
也就是说消息是平均分配给每个消费者，并没有考虑到消费者的处理能力。这样显然是有问题的。
## 2.3. 发布/订阅
实际为一次发送多个消费者都能接收到同样的数据的过程  
发布订阅的模型如图：  
<img src="https://raw.githubusercontent.com/PeipengWang/picture/master/20220220131809379_13195.png" alt="" width="1287">
可以看到，在订阅模型中，多了一个exchange角色，而且过程略有变化：
Publisher：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）  
Exchange：交换机，图中的X。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有以下3种类型：  
Fanout：广播，将消息交给所有绑定到交换机的队列  
Direct：定向，把消息交给符合指定routing key 的队列  
Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列  
Consumer：消费者，与以前一样，订阅队列，没有变化  
Queue：消息队列也与以前一样，接收消息、缓存消息。  

### 2.3.1. Fanout（广播）模式  
#### 2.3.1.1. 配置消费者的配置类  
声明交换机和消息队列，并将交换机和消息队列进行绑定  
```
package cn.itcast.mq.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FanoutConfig {
    /**
     * 声明交换机
     * @return Fanout类型交换机
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("itcast.fanout");
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```
配置完成后，可以发现名为itcast.fanout的交换机绑定了两个队列，分别为fanout.queue1和fanout.queue2的两个队列。，关闭项目后依然存在。  
<img src="https://raw.githubusercontent.com/PeipengWang/picture/master/20220220135606841_21350.png" alt="" width="767">

#### 2.3.1.2. 消息发送
在publisher服务的SpringAmqpTest类中添加测试方法：  
```
@Test
public void testFanoutExchange() {
    // 队列名称
    String exchangeName = "itcast.fanout";
    // 消息
    String message = "hello, everyone!";
    rabbitTemplate.convertAndSend(exchangeName, "", message);
}
```
#### 2.3.1.3. 消息接收    
在consumer服务的SpringRabbitListener中添加两个方法，作为消费者：  
```
@RabbitListener(queues = "fanout.queue1")  
public void listenFanoutQueue1(String msg) {
    System.out.println("消费者1接收到Fanout消息：【" + msg + "】");
}

@RabbitListener(queues = "fanout.queue2")
public void listenFanoutQueue2(String msg) {
    System.out.println("消费者2接收到Fanout消息：【" + msg + "】");
}
```
#### 2.3.1.4. 总结  
交换机的作用是什么？  
接收publisher发送的消息  
将消息按照规则路由到与之绑定的队列  
不能缓存消息，路由失败，消息丢失  
FanoutExchange的会将消息路由到每个绑定的队列  
声明队列、交换机、绑定关系的Bean是什么？  

Queue  
FanoutExchange  
Binding  

### 2.3.2. Direct模式  
在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。  

在Direct模型下：
队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）
消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。
Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息
<img src="https://raw.githubusercontent.com/PeipengWang/picture/master/20220220142211105_6575.png" alt="" width="1127">

#### 2.3.2.1. 接收消息
基于注解声明队列和交换机  
基于@Bean的方式声明队列和交换机比较麻烦，Spring还提供了基于注解方式来声明。  
在consumer的SpringRabbitListener中添加两个消费者，同时基于注解来声明队列和交换机：    
```
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue2"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "yellow"}
))
public void listenDirectQueue2(String msg){
    System.out.println("消费者接收到direct.queue2的消息：【" + msg + "】");
}
```
#### 2.3.2.2. 消息发送
在publisher服务的SpringAmqpTest类中添加测试方法：  
```
@Test
public void testSendDirectExchange() {
    // 交换机名称
    String exchangeName = "itcast.direct";
    // 消息
    String message = "红色警报！日本乱排核废水，导致海洋生物变异，惊现哥斯拉！";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "red", message);
}
```
对于消息发送来说，会向绑定itcast.direct的路由器的所有消息队列发数据，但最终消费者只能从key为red的消息队列获取数据  
### 2.3.3. Topic模式
Topic类型的Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key 的时候使用通配符！  
Routingkey 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： item.insert  
通配符规则：  
#：匹配一个或多个词  
*：匹配不多不少恰好1个词  
举例：  
item.#：能够匹配item.spu.insert 或者 item.spu  
item.*：只能匹配item.spu  
<img src="https://raw.githubusercontent.com/PeipengWang/picture/master/20220220145559813_953.png" alt="" width="1006">
解释：  

Queue1：绑定的是china.# ，因此凡是以 china.开头的routing key 都会被匹配到。包括china.news和china.weather  
Queue2：绑定的是#.news ，因此凡是以 .news结尾的 routing key 都会被匹配。包括china.news和japan.news  
案例需求：  
实现思路如下：  
并利用@RabbitListener声明Exchange、Queue、RoutingKey  
在consumer服务中，编写两个消费者方法，分别监听topic.queue1和topic.queue2  
在publisher中编写测试方法，向itcast. topic发送消息  
#### 2.3.3.1. 消息发送  
在publisher服务的SpringAmqpTest类中添加测试方法：  
```
/**
     * topicExchange
     */
@Test
public void testSendTopicExchange() {
    // 交换机名称
    String exchangeName = "itcast.topic";
    // 消息
    String message = "喜报！孙悟空大战哥斯拉，胜!";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "china.news", message);
}
```
#### 2.3.3.2. 消息接收  
在consumer服务的SpringRabbitListener中添加方法：  
```
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue1"),
    exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void listenTopicQueue1(String msg){
    System.out.println("消费者接收到topic.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue2"),
    exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void listenTopicQueue2(String msg){
    System.out.println("消费者接收到topic.queue2的消息：【" + msg + "】");
}
```
总结  
描述下Direct交换机与Topic交换机的差异？  

Topic交换机接收的消息RoutingKey必须是多个单词，以 **.** 分割  
Topic交换机与队列绑定时的bindingKey可以指定通配符  
#：代表0个或多个词  
*：代表1个词  

