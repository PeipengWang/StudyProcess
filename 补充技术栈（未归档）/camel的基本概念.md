## 一，什么是apache camel？

apache camel官网：https://camel.apache.org/

apache camel 是轻量级ESB框架（[什么是ESB框架？](https://zhuanlan.zhihu.com/p/97815422)）

它有几个比较重要的概念就是：

1. endpoint,所谓的endpoint,就是一种可以接收或发送数据的组件。可以支持多种协议，如jms,http,file等。
2. processor,它是用来处理具体业务逻辑的组件。
3. route,用来路由，指示数据从哪里来到哪里去，中间用哪个processor处理。

而processor之间用exchange对象来传送数据，有点像jms,通俗一点就像上学时传的小纸条，

架构图如下

![img](E:/%E7%AC%94%E8%AE%B0/StudyProcess/%E8%A1%A5%E5%85%85%E6%8A%80%E6%9C%AF%E6%A0%88%EF%BC%88%E6%9C%AA%E5%BD%92%E6%A1%A3%EF%BC%89/assets/544318-20160222215454239-1499086189.png)

camel就是企业信息集成框架，它提供了很多简单好用而又强大的组件，用户可以根据场景来选择不同的EIP（企业集成模式）来实现自己的需求，以响应快速变化的业务。可以把它当成企业信息总线（ESB）的轻量级实现。

camel是一款基于规则快速实现消息流转的开发组件，集成该组件后，你的程序可以编写最少的代码实现复杂的消息在不同的协议规则间流转。

例如：

程序实现从Ftp获得.xml文件，然后将收到的文件内容值转换后，发送到Jms Queue中，并且将Request写入到数据库log表。

Ftp组件->Jms组件->Db组件

只需要短短的几行代码就可以实现这样一个功能，但是如果用其他框架一个个功能的写，将会有非常多的代码量并且可能会出现一些纰漏，而camel已经将这些功能都封装在camel组件中了，节省开发成本。

from("ftp://xxxxxxxxxxxxx").bean("bean:JmsQueueCovertBean?method=convert").to("jms://xxxxxxxxxxx")..setBody(simple("insert into xxxxxxxxxxx")).to("jdbc:testdb");

当前热门的EIP集成框架分别有：Spring Integration、Mule ESB、Apache Camel。



## 二，CAMEL可以做什么

### 1，APACHE CAMEL简介

Apache camel 是一个基于EIP的开源框架。实现了EIP定义的一些不同应用系统之间的消息传输模型，包括常见的Point2Point、Pub/Sub模型。

**Camel的消息传递系统（Message System）2:**

- **终端（Message Endpoint）**：可以是异构的业务系统，都需要提供Endpoint实现集成。
- **通道（Message Channel）**：两个应用之间进行信息通讯的通道。
- **消息（Message）**：Endpoint之间交互的标准化单位。
- **路由（Message Router）**：根据一定的条件，将消息传递给不同的过滤器以实现对单个处理步骤的解耦。
- **转换器（Message Translator）**：消息在传输过程中的转换和数据映射，包括报文格式转换和内容转换映射。
- **管道和过滤器（Pipes & Filters）**：在保持独立性和灵活性的基础上，对复杂的消息进行处理。

### 2，CAMEL的应用场景

- **消息汇聚**：比如将来自不同服务器的数据，有ActiveMQ、RabbitMQ、WebService等的数据合成报表。
- **消息分发**：将消息从消息生产者转发给消息接收者，分发方式分为两种：顺序分发&并行分发。

```java
from("amqp:queue:order")
.to("uri:validateBean", "uri:handleBean", "uri:emailBean");

from("amqp:queue:order")
.multicast()
.to("uri:validateBean", "uri:handleBean", "uri:emailBean")
```

- **消息转换**：将消息内容进行转换，比如xml转为json格式。

```java
from("amqp:queue:order")
.process(new XmlToJsonProcessor())
.to("bean:orderHandler");
```

- **规则引擎**：可以使用Spring XML配置或DSL来定义route。同时camel提供了大量内置Processor，用于逻辑运算、过滤等，这样更容易灵活的管理route。

```xml
<route>
    <from uri="amqp:queue:order"/>
    <multicast>
        <to uri="uri:validateBean"/>
        <to uri="uri:handleBean"/>
        <to uri="uri:emailBean"/>
    </multicast>
</route>
```

```java
from("amqp:queue:order")
.filter(header("foo")
.isEqualTo("bar"))
.choice()
.when(xpath("/person/city = &#39;London&#39;"))
  .to("file:target/messages/uk")
.otherwise()
  .to("file:target/messages/others");
```

### 3，CAMEL的核心要素

![img](E:/%E7%AC%94%E8%AE%B0/StudyProcess/%E8%A1%A5%E5%85%85%E6%8A%80%E6%9C%AF%E6%A0%88%EF%BC%88%E6%9C%AA%E5%BD%92%E6%A1%A3%EF%BC%89/assets/1923677-20220209102149911-259075415.png)

Camel有以下五要素：

- Endpoint：用于收发消息。
- Exchange：消息本体。
- Processor：消息处理器。
- Routing：路由规则。
- Service：Camel基础概念。

### 4，ENDPOINT

- Endpoint是Camel与其他系统进行通信的设定点。
- Camel自身提供了广泛的通信协议支持，例如：RPC协议、HTTP协议、FTP协议……
- Camel中的Endpoint使用URI描述对目标系统的通信。
- 对Endpoint实例的创建通过对Camel中org.apche.camel.Component接口的实现来实现的。
- Camel通过Plug方式提供对各种协议的Endpoint支持，如果需要使用某种Endpoint，需要引入响应的plug。例如要使用Camel对Netty4-Endpoint的支持，要引入camel-netty4的依赖包。

### 5，EXCHANGE

![img](E:/%E7%AC%94%E8%AE%B0/StudyProcess/%E8%A1%A5%E5%85%85%E6%8A%80%E6%9C%AF%E6%A0%88%EF%BC%88%E6%9C%AA%E5%BD%92%E6%A1%A3%EF%BC%89/assets/1923677-20220209102253526-1858029753.png)

- **Properties**：Exchange对象贯穿整个路由执行过程中的控制端点、处理器甚至还有表达式、路由条件判断。为了让这些元素能够共享一些开发人员自定义的参数配置信息，Exchange以K-V结构提供了这样的参数配置信息存储方式。
- **Patterns**：Exchange中的pattern属性非常重要，它的全称是：ExchangePattern（交换器工作模式）。其实现是一个枚举类型：org.apache.camel.ExchangePattern。可以使用的值包括：InOnly, RobustInOnly, InOut, InOptionalOut, OutOnly, RobustOutOnly, OutIn, OutOptionalIn。从Camel官方已公布的文档来看，这个属性描述了Exchange中消息的传播方式。
- **Message IN/OUT**：当Endpoint和Processor、Processor和Processor间的Message在Exchange中传递时，Exchange会自动将上一个元素的输出作为这个元素的输入使用。



### 6，PROCESSOR

**Processor**用于接受从Endpoint、Routing或者另一个Processor的Exchange中传来的消息，并进行处理。
Camel核心包和各个Plugin组件都提供了很多Processor的实现，开发人员也可以通过实现org.apache.camel.Processor接口自定义Processor。

```java
// 一个自定义处理器的实现
public class OtherProcessor implements Processor {
    @Override
    public void process(Exchange exchange) throws Exception {
        Message message = exchange.getIn();
        String body = message.getBody().toString();
        //===============
        // 您可以在这里进行数据格式转换
        // 并且将结果存储到out message中
        //===============
        // 存入到exchange的out区域
        if(exchange.getPattern() == ExchangePattern.InOut) {
            Message outMessage = exchange.getOut();
            outMessage.setBody(body + " || other out");
        }
    }
}
```

### 7，ROUTING

**Routing**用于处理Endpoint和Processor之间、Processor和Processor之间的路由跳转。
Camel中支持的路由规则非常丰富，包括基于内容、接收者列表、循环动态路由等。、

![img](E:/%E7%AC%94%E8%AE%B0/StudyProcess/%E8%A1%A5%E5%85%85%E6%8A%80%E6%9C%AF%E6%A0%88%EF%BC%88%E6%9C%AA%E5%BD%92%E6%A1%A3%EF%BC%89/assets/1923677-20220209102356112-345261948.png)

### 8，SERVICE

在Apache Camel中有一个比Endpoint、Component、CamelContext等元素更基础的概念元素：Service。
包括Endpoint、Component、CamelContext等元素在内的大多数工作在Camel中的元素，都是一个一个的Service。
Camel应用程序中的每一个Service都是独立运行的，各个Service的关联衔接通过CamelContext上下文对象完成。每一个Service通过调用start()方法被**并参与到Camel应用程序的工作中，直到它的stop()方法被调用。也就是说，每个Service都有独立的生命周期。

### 9，CAMELCONTEXT上下文

**CamelContext**横跨了Camel服务的整个生命周期，并且为Camel服务的工作环境提供支撑。

## 三，代码实践

### 1，引入相关Jar包

```java
	<!-- apache camel -->
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-core</artifactId>
            <version>2.24.2</version>
        </dependency>
        <!-- apache camel 集成 activemq中间件 -->
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-camel</artifactId>
            <version>5.15.4</version>
        </dependency>
```

### 2，传输文件到消息件

```java
public static void main(String[] args) throws Exception {
        DefaultCamelContext context = new DefaultCamelContext();
        ActiveMQConnectionFactory connectionFactory =
                new ActiveMQConnectionFactory("admin","admin","http://172.16.2.221:8161");
        context.addComponent("activemq", JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("file:input_box?noop=true")
                        .to("activemq:queue:my_queue");
            }
        });
        context.start();
}
```

### 3，传输对象到消息中间件

```java
public static void main(String[] args) throws Exception {
        DefaultCamelContext context = new DefaultCamelContext();
        ActiveMQConnectionFactory connectionFactory =
                new ActiveMQConnectionFactory("admin","admin","tcp://172.16.2.221:61616");
        context.addComponent("activemq", JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("direct:start")
                        .to("activemq:queue:my_queue");
            }
        });
        context.start();
        ProducerTemplate producerTemplate = context.createProducerTemplate();
        producerTemplate.sendBody("direct:start","测试消息");
    }
```

### 4，生产者和消费者示例

process是一个处理器，可以处理消费者消费消息之前的消息。

```java
public static void main(String[] args) throws Exception {
        DefaultCamelContext context = new DefaultCamelContext();
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("direct:start")
                        .process(new Processor() {
                            @Override
                            public void process(Exchange exchange) throws Exception {
                                System.out.println("进入消息处理器...");
                                //提供者发送的消息
                                String msg = exchange.getIn().getBody(String.class);
                                msg = msg + "-By FanJiangFeng";
                                System.out.println("消息被我修改成："+msg);
                                //重新发送
                                exchange.getOut().setBody(msg);
                            }
                        })
                        .to("seda:end");
            }
        });

        context.start();
        //提供者
        ProducerTemplate producerTemplate = context.createProducerTemplate();
        producerTemplate.sendBody("direct:start","Hello Everyone");

        //消费者
        ConsumerTemplate consumerTemplate = context.createConsumerTemplate();
        String message = consumerTemplate.receiveBody("seda:end", String.class);

        System.out.println("消费者取出的消息："+message);

    }
```

### 5，消息生产者生产的数据为数据库中sql查询到的数据

需要Jar包

```xml
<!-- apache camel 集成 数据库 -->
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-jdbc</artifactId>
            <version>2.22.1</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
```

测试类

```java
public static void main(String[] args) throws Exception {
        MysqlDataSource dataSource = new MysqlDataSource();
        dataSource.setURL("jdbc:mysql://localhost:3306/test");
        dataSource.setUser("root");
        dataSource.setPassword("1234");

        SimpleRegistry simpleRegistry = new SimpleRegistry();
        simpleRegistry.put("myDataSource",dataSource);

        DefaultCamelContext context = new DefaultCamelContext(simpleRegistry);
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("direct:start")
                        .to("jdbc:myDataSource")
                        .bean(new ResultHandlerTest(),"printResult");
            }
        });

        context.start();
        //生产者
        ProducerTemplate producerTemplate = context.createProducerTemplate();
        producerTemplate.sendBody("direct:start","select * from user");
    }

    public static class ResultHandlerTest{
        private void printResult(List list){
            for(int i=0;i<list.size();i++){
                System.out.println(list.get(i));
            }
        }
    }
```

### 6，消息生产者发送消息，由某个类的某个方法进行消费消息（第一种方式）

消息生产者发送消息，由某个类的某个方法进行消费消息（此方法必须为public方法，否则报方法找不到的异常）

```java
public class CallMethodUsingClassComponent {

    public void consumerMethod(String message){
        System.out.println("消费者方法接收消息："+ message);
    }

    public static void main(String[] args) throws Exception{
        DefaultCamelContext context = new DefaultCamelContext();
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("direct:start")
                        .to("class:com.ftx.camel.test.CallMethodUsingClassComponent?method=consumerMethod");
            }
        });
        context.start();
        //创建消息生产者
        ProducerTemplate producerTemplate = context.createProducerTemplate();
        producerTemplate.sendBody("direct:start","测试消息");
    }
}
```

### 7,消息生产者发送消息，由某个类的某个方法进行消费消息(第二种方式)

此方法必须为public方法，否则报方法找不到的异常

```java
public class CallMethodUsingClassComponent2 {

    public void consumerMethod(String message){
        System.out.println("消费者方法接收消息："+ message);
    }

    public static void main(String[] args) throws Exception{

        CallMethodUsingClassComponent2 callMethodUsingClassComponent = new CallMethodUsingClassComponent2();
        SimpleRegistry registry = new SimpleRegistry();
        registry.put("myService",callMethodUsingClassComponent);

        DefaultCamelContext context = new DefaultCamelContext(registry);
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("direct:start")
                        .to("bean:myService?method=consumerMethod");
            }
        });
        context.start();
        //创建消息生产者
        ProducerTemplate producerTemplate = context.createProducerTemplate();
        producerTemplate.sendBody("direct:start","测试消息");
    }
}
```

### 8，消费者从activeMQ中消费消息

```java
public class ActiveMQConsumer {
    public static void main(String[] args) throws Exception {
        DefaultCamelContext context = new DefaultCamelContext();
        ActiveMQConnectionFactory connectionFactory =
                new ActiveMQConnectionFactory("admin","admin","tcp://172.16.2.221:61616");
        context.addComponent("activemq", JmsComponent.jmsComponentAutoAcknowledge(connectionFactory));
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("activemq:queue:my_queue").to("seda:end");
            }
        });
        context.start();
        ConsumerTemplate consumerTemplate = context.createConsumerTemplate();
        String message = consumerTemplate.receiveBody("seda:end", String.class);
        System.out.println(message);
    }
}
```

