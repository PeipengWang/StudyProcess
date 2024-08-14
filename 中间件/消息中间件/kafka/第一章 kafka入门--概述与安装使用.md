# 第1章 Kafka入门

## 1.1 概述

### 1.1.1 初识Kafka

Kafka是一个由Scala和Java语言开发的，经典高吞吐量的分布式消息发布和订阅系统，也是大数据技术领域中用作数据交换的核心组件之一。以高吞吐，低延迟，高伸缩，高可靠性，高并发，且社区活跃度高等特性，从而备受广大技术组织的喜爱。

2010年，Linkedin公司为了解决消息传输过程中由各种缺陷导致的阻塞、服务无法访问等问题，主导开发了一款分布式消息日志传输系统。主导开发的首席架构师Jay Kreps因为喜欢写出《变形记》的西方表现主义文学先驱小说家Jay Kafka，所以给这个消息系统起了一个很酷，却和软件系统特性无关的名称Kafka。

因为备受技术组织的喜爱，2011年，Kafka软件被捐献给Apache基金会，并于7月被纳入Apache软件基金会孵化器项目进行孵化。2012年10月，Kafka从孵化器项目中毕业，转成Apache的顶级项目。由独立的消息日志传输系统转型为开源分布式事件流处理平台系统，被数千家公司用于高性能数据管道、流分析、数据集成和关键任务应用程序。

官网地址：[https://kafka.apache.org/](https://www.scala-lang.org/)

![image-20240417220808196](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240417220808196.png) 

### 1.1.2 消息队列

Kafka软件最初的设计就是专门用于数据传输的消息系统，类似功能的软件有RabbitMQ、ActiveMQ、RocketMQ等。这些软件名称中的MQ是英文单词Message Queue的简称，也就是所谓的消息队列的意思。这些软件的核心功能是传输数据，而Java中如果想要实现数据传输功能，那么这个软件一般需要遵循Java消息服务技术规范JMS（Java Message Service）。前面提到的ActiveMQ软件就完全遵循了JMS技术规范，而RabbitMQ是遵循了类似JMS规范并兼容JMS规范的跨平台的AMQP（Advanced Message Queuing Protocol）规范。除了上面描述的JMS，AMQP外，还有一种用于物联网小型设备之间传输消息的MQTT通讯协议。

Kafka拥有作为一个消息系统应该具备的功能，但是却有着独特的设计。可以这样说，Kafka借鉴了JMS规范的思想，但是却并没有完全遵循JMS规范。这也恰恰是软件名称为Kafka，而不是KafkaMQ的原因。

由上可知，无论学习哪一种消息传输系统，JMS规范都是大家应该首先了解的。所以咱们这里就对JMS规范做一个简单的介绍：

 JMS是Java平台的消息中间件通用规范，定义了主要用于消息中间件的标准接口。如果不是很理解这个概念，可以简单地将JMS类比为Java和数据库之间的JDBC规范。Java应用程序根据JDBC规范种的接口访问关系型数据库，而每个关系型数据库厂商可以根据JDBC接口来实现具体的访问规则。JMS定义的就是系统和系统之间传输消息的接口。

为了实现系统和系统之间的数据传输，JMS规范中定义很多用于通信的组件：

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps2.jpg) 

 JMS Provider：JMS消息提供者。其实就是实现JMS接口和规范的消息中间件，也就是我们提供消息服务的软件系统，比如RabbitMQ、ActiveMQ、Kafka。

 JMS Message：JMS消息。这里的消息指的就是数据。一般采用Java数据模型进行封装，其中包含消息头，消息属性和消息主体内容。

JMS Producer：JMS消息生产者。所谓的生产者，就是生产数据的客户端应用程序，这些应用通过JMS接口发送JMS消息。

JMS Consumer：JMS消息消费者。所谓的消费者，就是从消息提供者（***JMS*** ***Provider***）中获取数据的客户端应用程序，这些应用通过JMS接口接收JMS消息。

JMS支持两种消息发送和接收模型：一种是P2P（Peer-to-Peer）点对点模型，另外一种是发布/订阅（Publish/Subscribe）模型。

 P2P模型 ：P2P模型是基于队列的，消息生产者将数据发送到消息队列中，消息消费者从消息队列中接收消息。因为队列的存在，消息的异步传输成为可能。P2P模型的规定就是每一个消息数据，只有一个消费者，当发送者发送消息以后，不管接收者有没有运行都不影响消息发布到队列中。接收者在成功接收消息后会向发送者发送接收成功的消息

发布 / 订阅模型 ：所谓得发布订阅模型就是事先将传输的数据进行分类，我们管这个数据的分类称之为主题（Topic）。也就是说，生产者发送消息时，会根据主题进行发送。比如咱们的消息中有一个分类是NBA，那么生产者在生产消息时，就可以将NBA篮球消息数据发送到NBA主题中，这样，对NBA消息主题感兴趣的消费者就可以申请订阅NBA主题，然后从该主题中获取消息。这样，也就是说一个消息，是允许被多个消费者同时消费的。这里生产者发送消息，我们称之为发布消息，而消费者从主题中获取消息，我们就称之为订阅消息。Kafka采用就是这种模型。

### **1.1.3** 生产者-消费者模式

生产者-消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通信，而通过阻塞队列来进行通信，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个消息缓冲区，平衡了生产者和消费者的处理能力。在数据传输过程中，起到了一个削弱峰值的作用，也就是我们经常说到的削峰。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps3.jpg) 

图形中的缓冲区就是用来给生产者和消费者解耦的。在单点环境中，我们一般会采用阻塞式队列实现这个缓冲区。而在分布式环境中，一般会采用第三方软件实现缓冲区，这个第三方软件我们一般称之为中间件。纵观大多数应用场景，解耦合最常用的方式就是增加中间件。

遵循JMS规范的消息传输软件（RabbitMQ、ActiveMQ、Kafka、RocketMQ），我们一般就称之为消息中间件。使用软件的目的本质上也就是为了降低消息生产者和消费者之间的耦合性。提升消息的传输效率。

### **1.1.4** 息中间件对比

|  特性               |  ActiveMQ                    |  RabbitMQ                      |  RocketMQ                  | Kafka                                            |
| ----------------------- | ---------------------------------- | ---------------------------------- | -------------------------------- | ------------------------------------------------------- |
| 单机吞吐量              | 万级，比RocketMQ,Kafka低一个数量级 | 万级，比RocketMQ,Kafka低一个数量级 | 10万级，支持高吞吐               | 10万级，支持高吞吐                                      |
| Topic数量对吞吐量的影响 |                                    |                                    | Topic可以达到几百/几千量级       | Topic可以达到几百量级，如果更多的话，吞吐量会大幅度下降 |
| 时效性                  | ms级                               | 微秒级别，延迟最低                 | ms级                             | ms级                                                    |
| 可用性                  | 高，基于主从架构实现高可用         | 高，基于主从架构实现高可用         | 非常高，分布式架构               | 非常高，分布式架构                                      |
| 消息可靠性              | 有较低的概率丢失数据               | 基本不丢                           | 经过参数优化配置，可以做到0丢失  | 经过参数优化配置，可以做到0丢失                         |
| 功能支持                | MQ领域的功能极其完备               | 并发能力强，性能极好，延时很低     | MQ功能较为完善，分布式，扩展性好 | 功能较为简单，支持简单的MQ功能，在大数据领域被广泛使用  |
| 其他                    | 很早的软件，社区不是很活跃         | 开源，稳定，社区活跃度高           | 阿里开发，社区活跃度不高         | 开源，高吞吐量，社区活跃度极高                          |

通过上面各种消息中间件的对比，大概可以了解，在大数据场景中我们主要采用kafka作为消息中间件，而在JaveEE开发中我们主要采用ActiveMQ、RabbitMQ、RocketMQ作为消息中间件。如果将JavaEE和大数据在项目中进行融合的话，那么Kafka其实是一个不错的选择。

### 1.1.5 ZooKeeper

ZooKeeper是一个开放源码的分布式应用程序协调服务软件。在当前的Web软件开发中，多节点分布式的架构设计已经成为必然，那么如何保证架构中不同的节点所运行的环境，系统配置是相同的，就是一个非常重要的话题。一般情况下，我们会采用独立的第三方软件保存分布式系统中的全局环境信息以及系统配置信息，这样系统中的每一个节点在运行时就可以从第三方软件中获取一致的数据。也就是说通过这个第三方软件来协调分布式各个节点之间的环境以及配置信息。Kafka软件是一个分布式事件流处理平台系统，底层采用分布式的架构设计，就是说，也存在多个服务节点，多个节点之间Kafka就是采用ZooKeeper来实现协调调度的。

ZooKeeper的核心作用：

l ZooKeeper的数据存储结构可以简单地理解为一个Tree结构，而Tree结构上的每一个节点可以用于存储数据，所以一般情况下，我们可以将分布式系统的元数据（环境信息以及系统配置信息）保存在ZooKeeper节点中。

l ZooKeeper创建数据节点时，会根据业务场景创建临时节点或永久（持久）节点。永久节点就是无论客户端是否连接上ZooKeeper都一直存在的节点，而临时节点指的是客户端连接时创建，断开连接后删除的节点。同时，ZooKeeper也提供了Watch（监控）机制用于监控节点的变化，然后通知对应的客户端进行相应的变化。Kafka软件中就内置了ZooKeeper的客户端，用于进行ZooKeeper的连接和通信。

其实，Kafka作为一个独立的分布式消息传输系统，还需要第三方软件进行节点间的协调调度，不能实现自我管理，无形中就导致Kafka和其他软件之间形成了耦合性，制约了Kafka软件的发展，所以从Kafka 2.8.X版本开始，Kafka就尝试增加了Raft算法实现节点间的协调管理，来代替ZooKeeper。不过Kafka官方不推荐此方式应用在生产环境中，计划在Kafka 4.X版本中完全移除ZooKeeper，让我们拭目以待。

ZooKeeper的核心作用：

 ZooKeeper的数据存储结构可以简单地理解为一个Tree结构，而Tree结构上的每一个节点可以用于存储数据，所以一般情况下，我们可以将分布式系统的元数据（环境信息以及系统配置信息）保存在ZooKeeper节点中。

 ZooKeeper创建数据节点时，会根据业务场景创建临时节点或永久（持久）节点。永久节点就是无论客户端是否连接上ZooKeeper都一直存在的节点，而临时节点指的是客户端连接时创建，断开连接后删除的节点。同时，ZooKeeper也提供了Watch（监控）机制用于监控节点的变化，然后通知对应的客户端进行相应的变化。Kafka软件中就内置了ZooKeeper的客户端，用于进行ZooKeeper的连接和通信。

其实，Kafka作为一个独立的分布式消息传输系统，还需要第三方软件进行节点间的协调调度，不能实现自我管理，无形中就导致Kafka和其他软件之间形成了耦合性，制约了Kafka软件的发展，所以从Kafka 2.8.X版本开始，Kafka就尝试增加了Raft算法实现节点间的协调管理，来代替ZooKeeper。不过Kafka官方不推荐此方式应用在生产环境中，计划在Kafka 4.X版本中完全移除ZooKeeper，让我们拭目以待。

## 1.1 快速上手

### 1.1.1 环境安装

作为开源分布式事件流处理平台，Kafka分布式软件环境的安装相对比较复杂，不利于Kafka软件的入门学习和练习。所以我们这里先搭建相对比较简单的windows单机环境，让初学者快速掌握软件的基本原理和用法，后面的课程中，我们再深入学习Kafka软件在生产环境中的安装和使用。

#### 1.2.1.1安装Java8（略）

当前Java软件开发中，主流的版本就是Java  8，而Kafka 3.X官方建议Java版本更新至Java11，但是Java8依然可用。未来Kafka 4.X版本会完全弃用Java8，不过，咱们当前学习的Kafka版本为3.6.1版本，所以使用Java8即可，无需升级。

Kafka的绝大数代码都是Scala语言编写的，而Scala语言本身就是基于Java语言开发的，并且由于Kafka内置了Scala语言包，所以Kafka是可以直接运行在JVM上的，无需安装其他软件。你能看到这个课件，相信你肯定已经安装Java8了，基本的环境变量也应该配置好了，所以此处安装过程省略。

直接安装即可

```
yum install -y java-1.8.0-openjdk-devel.x86_64
```

#### 1.2.1.2 安装Kafka

下载软件安装包：kafka_2.12-3.6.1.tgz，下载地址：https://kafka.apache.org/downloads

 这里的3.6.1，是Kafka软件的版本。截至到2023年12月24日，Kafka最新版本为3.6.1。

 2.12是对应的Scala开发语言版本。Scala2.12和Java8是兼容的，所以可以直接使用。

 tgz是一种linux系统中常见的压缩文件格式，类似与windows系统的zip和rar格式。所以Windows环境中可以直接使用压缩工具进行解压缩。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps4.jpg) 

解压文件：kafka_2.12-3.6.1.tgz，解压目录为非系统盘的根目录，比如e:/

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps5.jpg) 

为了访问方便，可以将解压后的文件目录改为kafka， 更改后的文件目录结构如下：

| bin         | linux系统下可执行脚本文件   |
| ----------- | --------------------------- |
| bin/windows | windows系统下可执行脚本文件 |
| config      | 配置文件                    |
| libs        | 依赖类库                    |
| licenses    | 许可信息                    |
| site-docs   | 文档                        |
| logs        | 服务日志                    |

#### 1.2.1.3 启动ZooKeeper

当前版本Kafka软件内部依然依赖ZooKeeper进行多节点协调调度，所以启动Kafka软件之前，需要先启动ZooKeeper软件。不过因为Kafka软件本身内置了ZooKeeper软件，所以无需额外安装ZooKeeper软件，直接调用脚本命令启动即可。具体操作步骤如下：

 进入Kafka解压缩文件夹的config目录，修改zookeeper.properties配置文件
```
# the directory where the snapshot is stored.
# 修改dataDir配置，用于设置ZooKeeper数据存储位置，该路径如果不存在会自动创建。
dataDir=E:/kafka_2.12-3.6.1/data/zk
```
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

出现如下界面，ZooKeeper启动成功。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps8.jpg) 

为了操作方便，也可以在kafka解压缩后的目录中，创建脚本文件zk.cmd。

调用启动命令，且同时指定配置文件。
```

# Start the Kafka broker service
$ bin/kafka-server-start.sh config/server.properties
所有服务成功启动后，您将拥有一个正在运行并可供使用的基本 Kafka 环境。
```
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps11.jpg) 



上面两个如果直接在窗口输入命令，在退出窗口时就关闭了，因此，可以在后台跑

```
nohup sh bin/zookeeper-server-start.sh config/zookeeper.properties > zoo.log 2>&1 &
nohup sh bin/kafka-server-start.sh config/server.properties > kafka.log 2>&1 &
```

### 1.1.2 消息主题

在消息发布/订阅（Publish/Subscribe）模型中，为了可以让消费者对感兴趣的消息进行消费，而不是对所有的数据进行消费，包括那些不感兴趣的消息，所以定义了主题（Topic）的概念，也就是说将不同的消息进行分类，分成不同的主题（Topic），然后消息生产者在生成消息时，就会向指定的主题（Topic）中发送。而消息消费者也可以订阅自己感兴趣的主题（Topic）并从中获取消息。

有很多种方式都可以操作Kafka消息中的主题（Topic）：命令行、第三方工具、Java API、自动创建。而对于初学者来讲，掌握基本的命令行操作是必要的。所以接下来，我们采用命令行进行操作。

#### 1.2.2.1创建主题

```
 bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

 Kafka是通过**kafka-topics.sh**指令文件进行消息主题操作的。其中包含了对主题的查询，创建，删除等功能。

调用指令创建主题时，需要传递多个参数，而且参数的前缀为两个横线。因为参数比较多，为了演示方便，这里我们只说明必须传递的参数，其他参数后面课程中会进行讲解

--bootstrap-server : 把当前的DOS窗口当成Kafka的客户端，那么进行操作前，就需要连接服务器，这里的参数就表示服务器的连接方式，因为我们在本机启动Kafka服务进程，且Kafka默认端口为9092，所以此处，后面接的参数值为***localhost:9092\**，用空格隔开

--create : 表示对主题的创建操作，是个操作参数，后面无需增加参数值

--topic : 主题的名称，后面接的参数值一般就是见名知意的字符串名称，类似于java中的字符串类型标识符名称，当然也可以使用数字，只不过最后还是当成数字字符串使用。
指令
![image-20240427230757865](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240427230757865.png)

#### 1.2.2.2 查询主题

DOS窗口输入指令，查看所有主题
Kafka是通过**kafka-topics.sh**文件进行消息主题操作的。其中包含了对主题的查询，创建，删除等功能。
**--bootstrap-server** : 把当前的DOS窗口当成Kafka的客户端，那么进行操作前，就需要连接服务器，这里的参数就表示服务器的连接方式，因为我们在本机启动Kafka服务进程，且Kafka默认端口为9092，所以此处，后面接的参数值为localhost:9092，用空格隔开
 **list** : 表示对所有主题的查询操作，是个操作参数，后面无需增加参数值

指令
```
/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```
 DOS窗口输入指令，查看指定主题信息

**--bootstrap-server** : 把当前的DOS窗口当成Kafka的客户端，那么进行操作前，就需要连接服务器，这里的参数就表示服务器的连接方式，因为我们在本机启动Kafka服务进程，且Kafka默认端口为9092，所以此处，后面接的参数值为localhost:9092，用空格隔开
**--describe** : 查看主题的详细信息
**--topic** : 查询的主题名称

指令
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic test
```
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps16.jpg) 

#### 1.2.2.3 修改主题
创建主题后，可能需要对某些参数进行修改，那么就需要使用指令进行操作。
DOS窗口输入指令，修改指定主题的参数

 Kafka是通过**kafka-topics.sh**文件进行消息主题操作的。其中包含了对主题的查询，创建，删除等功能。

**--bootstrap-server** : 把当前的DOS窗口当成Kafka的客户端，那么进行操作前，就需要连接服务器，这里的参数就表示服务器的连接方式，因为我们在本机启动Kafka服务进程，且Kafka默认端口为9092，所以此处，后面接的参数值为localhost:9092，用空格隔开  
**--alter** : 表示对所有主题的查询操作，是个操作参数，后面无需增加参数值  
**--topic** : 修改的主题名称  
**--partitions**: 修改的配置参数：分区数量  
 指令

```
kafka-topics.sh --bootstrap-server localhost:9092 --topic test --alter --partitions 2
```
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps17.jpg) 

#### 1.2.2.4删除主题

如果主题创建后不需要了，或创建的主题有问题，那么我们可以通过相应的指令删除主题。

 DOS窗口输入指令，删除指定名称的主题

Kafka是通过**kafka-topics.bat**文件进行消息主题操作的。其中包含了对主题的查询，创建，删除等功能。

**--bootstrap-server** : 把当前的DOS窗口当成Kafka的客户端，那么进行操作前，就需要连接服务器，这里的参数就表示服务器的连接方式，因为我们在本机启动Kafka服务进程，且Kafka默认端口为9092，所以此处，后面接的参数值为localhost:9092，用空格隔开

**--delete**: 表示对主题的删除操作，是个操作参数，后面无需增加参数值。默认情况下，删除操作是逻辑删除，也就是说数据存储的文件依然存在，但是通过指令查询不出来。如果想要直接删除，需要在server.properties文件中设置参数delete.topic.enable=true

**--topic** : 删除的主题名称

指令
```
kafka-topics.sh --bootstrap-server localhost:9092 --topic test --delete
```
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps18.jpg) 

**注意**：windows系统中由于权限或进程锁定的问题，删除topic会导致kafka服务节点异常关闭。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps19.jpg) 

请在后续的linux系统下演示此操作。

### **1.1.3** 生产数据

消息主题创建好了，就可以通过Kafka客户端向Kafka服务器的主题中发送消息了。Kafka生产者客户端并不是一个独立的软件系统，而是一套API接口，只要通过接口能连接Kafka并发送数据的组件我们都可以称之为Kafka生产者。下面我们就演示几种不同的方式：

#### **1.2.3.1命令行操作**

Kafka是通过**kafka-console-producer.sh**文件进行消息生产者操作的。

调用指令时，需要传递多个参数，而且参数的前缀为两个横线，因为参数比较多。为了演示方便，这里我们只说明必须传递的参数，其他参数后面课程中会进行讲解

**--bootstrap-server** : 把当前的DOS窗口当成Kafka的客户端，那么进行操作前，就需要连接服务器，这里的参数就表示服务器的连接方式，因为我们在本机启动Kafka服务进程，且Kafka默认端口为9092，所以此处，后面接的参数值为**\*localhost:9092\***，用空格隔开。早期版本的Kafka也可以通过 --broker-list参数进行连接，当前版本已经不推荐使用了。

***--topic*** : 主题的名称，后面接的参数值就是之前已经创建好的主题名称。

指令

```
 bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
>test 1
>test 2
>
```

**注意**：这里的数据需要回车后，才能真正将数据发送到Kafka服务器。



读取消息，打开另一个终端会话并运行控制台消费者客户端来读取您刚刚创建的事件：

```
[root@node1 kafka_2.13-3.6.2]# bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
test 1
test 2

```

#### **1.2.3.2工具操作**
有的时候，使用命令行进行操作还是有一些麻烦，并且操作起来也不是很直观，所以我们一般会采用一些小工具进行快速访问。这里我们介绍一个kafkatool_64bit.exe工具软件。软件的安装过程比较简单，根据提示默认安装即可，这里就不进行介绍了。

安装好以后，我们打开工具
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps23.jpg) 

点击左上角按钮File -> Add New Connection...建立连接

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps24.jpg) 

点击Test按钮测试

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps25.jpg) 
增加连接
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps26.jpg) 

按照下面的步骤，生产数据

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps27.jpg) 

增加成功后，点击绿色箭头按钮进行查询，工具会显示当前数据

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps28.jpg) 

#### **1.2.3.3 Java API**

一般情况下，我们也可以通过Java程序来生产数据，所以接下来，我们就演示一下IDEA中使用Kafka Java API来生产数据：

 修改pom.xml文件，增加Maven依赖 ，跟kafaka版本一致

```
<dependencies>
  <dependency>
​    <groupId>org.apache.kafka</groupId>
​    <artifactId>kafka-clients</artifactId>
​    <version>3.6.1</version>
  </dependency>
</dependencies>
```
添加main方法，并增加生产者代码 

```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.HashMap;
import java.util.Map;

public class KafkaProducerTest {
    public static void main(String[] args) {
        // 配置属性集合
        Map<String, Object> configMap = new HashMap<>();
        //  配置属性：Kafka服务器集群地址
        configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "43.143.251.77:9092");
        //  配置属性：Kafka生产的数据为KV对，所以在生产数据进行传输前需要分别对K,V进行对应的序列化操作
        configMap.put(
                ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer");
        configMap.put(
                ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer");
        //  创建Kafka生产者对象，建立Kafka连接
        //    构造对象时，需要传递配置参数
        KafkaProducer<String, String> producer = new KafkaProducer<>(configMap);
        //   准备数据,定义泛型
        //    构造对象时需要传递 【Topic主题名称】，【Key】，【Value】三个参数
        ProducerRecord<String, String> record = new ProducerRecord<>(
                " quickstart-events", "key1", "value1"
        );
        //  生产（发送）数据
        producer.send(record);
        //  关闭生产者连接
        producer.close();

    }
}

```
### **1.1.4** 消费数据

消息已经通过Kafka生产者客户端发送到Kafka服务器中了。那么此时，这个消息就会暂存在Kafka中，我们也就可以通过Kafka消费者客户端对服务器指定主题的消息进行消费了。

#### **1.2.4.1命令行操作**

DOS窗口输入指令，进入消费者控制台

Kafka是通过***kafka-console-consumer.sh***文件进行消息消费者操作的。

调用指令时，需要传递多个参数，而且参数的前缀为两个横线，因为参数比较多。为了演示方便，这里我们只说明必须传递的参数，其他参数后面课程中会进行讲解

```
kafka_2.13-3.6.2]# bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```

**--bootstrap-server** : 把当前的DOS窗口当成Kafka的客户端，那么进行操作前，就需要连接服务器，这里的参数就表示服务器的连接方式，因为我们在本机启动Kafka服务进程，且Kafka默认端口为9092，所以此处，后面接的参数值为***localhost:9092**，用空格隔开。早期版本的Kafka也可以通过 --broker-list参数进行连接，当前版本已经不推荐使用了。

**--topic** : 主题的名称，后面接的参数值就是之前已经创建好的主题名称。其实这个参数并不是必须传递的参数，因为如果不传递这个参数的话，那么消费者会消费所有主题的消息。如果传递这个参数，那么消费者只能消费到指定主题的消息数据。

**--from-beginning** : 从第一条数据开始消费，无参数值，是一个标记参数。默认情况下，消费者客户端连接上服务器后，是不会消费到连接之前所生产的数据的。也就意味着如果生产者客户端在消费者客户端连接前已经生产了数据，那么这部分数据消费者是无法正常消费到的。所以在实际环境中，应该是先启动消费者客户端，再启动生产者客户端，保证消费数据的完整性。增加参数后，Kafka就会从第一条数据开始消费，保证消息数据的完整性。

#### 1.2.4.2 Java API

一般情况下，我们可以通过Java程序来消费（获取）数据，所以接下来，我们就演示一下IDEA中Kafka Java API如何消费数据：

创建com.atguigu.kafka.test.KafkaConsumerTest类 

```
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.time.Duration;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

public class KafkaConsumerTest {

  public static void main(String[] args) {
    // 配置属性集合
    Map<String, Object> configMap = new HashMap<String, Object>();
   //  配置属性：Kafka集群地址
   configMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "43.143.251.77:9092");
   // 配置属性: Kafka传输的数据为KV对，所以需要对获取的数据分别进行反序列化
   configMap.put(
       ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
       "org.apache.kafka.common.serialization.StringDeserializer");
   configMap.put(
       ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
       "org.apache.kafka.common.serialization.StringDeserializer");
   //  配置属性: 读取数据的位置 ，取值为earliest（最早），latest（最晚）
   configMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"earliest");
   // 配置属性: 消费者组
    configMap.put("group.id", "atguigu");
   // 配置属性: 自动提交偏移量
   configMap.put("enable.auto.commit", "true");
   KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configMap);
   //  消费者订阅指定主题的数据
   consumer.subscribe(Collections.singletonList("quickstart-events"));
   while (true) {
     // 每隔100毫秒，抓取一次数据
     ConsumerRecords<String, String> records =
             consumer.poll(Duration.ofMillis(100));
     // 打印抓取的数据
     for (ConsumerRecord<String, String> record : records) {
       System.out.println("K = " + record.key() + ", V = " + record.value());
     }
   }

  }

}
```
### **1.1.5** 源码关联(可选)

将源码压缩包kafka-3.6.1-src.tgz解压缩到指定位置

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps38.jpg) 

Kafka3.6.1的源码需要使用JDK17和Scala2.13进行编译才能查看，所以需要进行安装

#### 1.2.5.1 安装Java17

(1) 再资料文件夹中双击安装包jdk-17_windows-x64_bin.exe

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps39.jpg) 

(2) 根据安装提示安装即可。

#### 1.2.5.2 安装Scala

(1) 进入Scala官方网站https://www.scala-lang.org/下载Scala压缩包scala-2.13.12.zip。

(2) 在IDEA中安装Scala插件

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps40.jpg) 

(3) 项目配置中关联Scala就可以了

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps41.jpg) 

#### 1.2.5.3 安装Gradle

(1) 进入Gradle官方网站https://gradle.org/releases/下载Gradle安装包，根据自己需要选择不同版本进行下载。下载后将Gradle文件解压到相应目录

(2) 新增系统环境GRADLE_HOME，指定gradle安装路径，并将%GRADLE_HOME%\bin添加到path中

(3) Gradle安装及环境变量配置完成之后，打开Windows的cmd命令窗口，输入gradle –version

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps42.jpg) 

(4) 在解压缩目录中打开命令行，依次执行gradle idea命令

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps43.jpg) 

(5) 在命令行中执行gradle build --exclude-task test命令

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps44.jpg) 

(6) 使用IDE工具IDEA打开该项目目录

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/wps45.jpg) 

### 1.1.6 设置允许远程连接

修改server.properties文件，例如如果kafka主机的ip是43.143.251.77

![image-20240428222352062](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428222352062.png)

### 1.1.7 总结

基本安装流程

解压kafaka

```
tar -xzf kafka_2.13-3.6.2.tgz
cd kafka_2.13-3.6.2
```

先启动Zookeper

```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

另一个会话启动kafka

```
bin/kafka-server-start.sh config/server.properties
```

当然也可以直接nohup方式启动

```
nohup sh bin/zookeeper-server-start.sh config/zookeeper.properties > zoo.log 2>&1 &
nohup sh bin/kafka-server-start.sh config/server.properties  >kafka.log 2>&1 &
```

创建一个主题

```
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

生产者将事件写入主题

```
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
```

消费者读取事件

```
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```



本章作为Kafka软件的入门章节，介绍了一些消息传输系统中的基本概念以及单机版Windows系统中Kafka软件的基本操作。如果仅从操作上，感觉Kafka和数据库的功能还是有点像的。比如：

数据库可以创建表保存数据，kafka可以创建主题保存消息。

Java客户端程序可以通过JDBC访问数据库：保存数据、修改数据、查询数据，kafka可以通过生产者客户端生产数据，通过消费者客户端消费数据。

从这几点来看，确实有相像的地方，但其实两者的本质并不一样：

数据库的本质是为了更好的组织和管理数据，所以关注点是如何设计更好的数据模型用于保存数据，保证核心的业务数据不丢失，这样才能准确地对数据进行操作。

Kafka的本质是为了高效地传输数据。所以软件的侧重点是如何优化传输的过程，让数据更快，更安全地进行系统间的传输。

通过以上的介绍，你会发现，两者的区别还是很大的，不能混为一谈。接下来的章节我们会给大家详细讲解Kafka在分布式环境中是如何高效地传输数据的。

