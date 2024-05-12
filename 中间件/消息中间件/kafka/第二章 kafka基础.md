# 第二章 Kafka基础

Kafka借鉴了JMS规范的思想，但是却并没有完全遵循JMS规范，因此从设计原理上，Kafka的内部也会有很多用于数据传输的组件对象，这些组件对象之间会形成关联，组合在一起实现高效的数据传输。所以接下来，我们就按照数据流转的过程详细讲一讲Kafka中的基础概念以及核心组件。

## 2.1 集群部署


### 2.1.1 解压文件

(1) 在磁盘根目录创建文件夹cluster，文件夹名称不要太长
(2) 将kafka安装包kafka-3.6.1-src.tgz解压缩到kafka文件夹

### 2.1.2 安装ZooKeeper

(1) 修改文件夹名为kafka-zookeeper
因为kafka内置了ZooKeeper软件，所以此处将解压缩的文件作为ZooKeeper软件使用。

### 2.1.3 安装Kafka
(1) 修改config/server.properties配置文件
(2) 文件夹中的配置文件server.properties为server1.properties、server2.properties 和 server3.properties
将文件内容中的broker.id=1分别改为broker.id=2，broker.id=3
将文件内容中的9091分别改为9092，9093（如果端口冲突，请重新设置）

这里包括两个地方：

listeners=PLAINTEXT://:9093

advertised.listeners=PLAINTEXT://43.143.251.77:9093

修改日志位置

```
log.dirs=/home/kafka/kafka_2.13-3.6.2_2/data
```

这个过程可以生成脚本（需要完善）
```
#!/bin/bash

# 复制并修改配置文件中的 broker.id 和端口号
copy_and_modify_config() {
    local original_file="$1"
    local copy_file="$2"

    # 复制配置文件
    cp "$original_file" "$copy_file"
    echo "Copied $original_file to $copy_file"

    # 修改配置文件中的 broker.id 和端口号
    sed -i 's/^broker.id=0$/broker.id=2/g' "$copy_file"
    sed -i 's/^broker.id=0$/broker.id=3/g' "$copy_file"
    sed -i 's/^9091$/9092/g' "$copy_file"
    sed -i 's/^9091$/9093/g' "$copy_file"
}

# 复制并修改 server1.properties
copy_and_modify_config "config/server.properties" "config/server1.properties"
echo "Modified server1.properties successfully!"

# 复制并修改 server2.properties
copy_and_modify_config "config/server.properties" "config/server2.properties"
echo "Modified server2.properties successfully!"

# 复制并修改 server3.properties
copy_and_modify_config "config/server.properties" "config/server3.properties"
echo "Modified server3.properties successfully!"

```
这个脚本首先定义了一个名为 copy_and_modify_config 的函数，该函数接受两个参数：原始配置文件和要复制到的新文件路径。在函数中，首先使用 cp 命令复制原始配置文件到新文件，然后使用 sed 命令修改新文件中的内容。最后，我们分别调用该函数来复制和修改 server.properties 到 server1.properties、server2.properties 和 server3.properties 文件。

要使用此脚本，只需将其保存为 .sh 文件，例如 copy-and-modify-config-files.sh，然后确保具有执行权限 (chmod +x copy-and-modify-config-files.sh)，最后运行它 (./copy-and-modify-config-files.sh)。执行完毕后，您将得到三个修改后的配置文件副本。



需要额外注意

```
zookeeper.connect=localhost:2181
```

这个参数是加入的zk集群，这个集群的位置就是将要连接的集群

### 2.1.4 封装启动脚本
启动 Kafka 集群需要多个步骤，包括启动 ZooKeeper 实例和 Kafka 服务器实例。以下是一个简单的示例脚本，可以用来启动 Kafka 集群。假设您的 Kafka 集群由3个 Kafka 服务器实例和1个 ZooKeeper 实例组成。
```
#!/bin/bash
# 启动 ZooKeeper
echo "Starting ZooKeeper..."
nohup sh bin/zookeeper-server-start.sh config/zookeeper.properties > /dev/null 2>&1 &
ZK_PID=$!
echo "ZooKeeper started with PID $ZK_PID"
# 等待一段时间以确保 ZooKeeper 启动完成
sleep 5
# 启动 Kafka 服务器实例
for i in {1..3}; do
    echo "Starting Kafka server $i..."
    nohup sh bin/kafka-server-start.sh config/server$i.properties > /dev/null 2>&1 &
    KAFKA_PID=$!
    echo "Kafka server $i started with PID $KAFKA_PID"
done
# 等待一段时间以确保 Kafka 服务器实例启动完成
sleep 10
echo "Kafka cluster started successfully!"
```
在此脚本中，假设您的 Kafka 服务器配置文件为 server1.properties、server2.properties 和 server3.properties，而 ZooKeeper 的配置文件为 zookeeper.properties。您需要根据实际情况调整这些文件的路径和内容。

要运行此脚本，只需将其保存为 .sh 文件，例如 start-kafka-cluster.sh，然后确保具有执行权限 (chmod +x start-kafka-cluster.sh)，最后运行它 (./start-kafka-cluster.sh)。

```
Starting ZooKeeper...
ZooKeeper started with PID 12075
Starting Kafka server 1...
Kafka server 1 started with PID 12201
Starting Kafka server 2...
Kafka server 2 started with PID 12202
Starting Kafka server 3...
Kafka server 3 started with PID 12203
Kafka cluster started successfully!
```

## 2.2 集群启动

### 2.2.1 相关概念

#### 2.2.1.1 代理：Broker

使用Kafka前，我们都会启动Kafka服务进程，这里的Kafka服务进程我们一般会称之为Kafka Broker或Kafka Server。因为Kafka是分布式消息系统，所以在实际的生产环境中，是需要多个服务进程形成集群提供消息服务的。所以每一个服务节点都是一个broker，而且在Kafka集群中，为了区分不同的服务节点，每一个broker都应该有一个不重复的全局ID，称之为broker.id，这个ID可以在kafka软件的配置文件server.properties中进行配置
\# 集群ID
broker.id=0
咱们的Kafka集群中每一个节点都有自己的ID，整数且唯一。

| 主机      | kafka-broker1 | kafka-broker2 | kafka-broker3 |
| --------- | ------------- | ------------- | ------------- |
| broker.id | 1             | 2             | 3             |

#### 2.2.1.2 控制器：Controller
Kafka是分布式消息传输系统，所以存在多个Broker服务节点，但是它的软件架构采用的是分布式系统中比较常见的主从（Master - Slave）架构，也就是说需要从多个Broker中找到一个用于管理整个Kafka集群的Master节点，这个节点，我们就称之为Controller。它是Apache Kafka的核心组件非常重要。它的主要作用是在Apache Zookeeper的帮助下管理和协调控制整个Kafka集群。

![image-20240428230758525](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428230758525.png) 

如果在运行过程中，Controller节点出现了故障，那么Kafka会依托于ZooKeeper软件选举其他的节点作为新的Controller，让Kafka集群实现高可用。

![image-20240428230822859](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428230822859.png) 

**Kafka集群中Controller的基本功能**：

**1、Broker管理**

监听 /brokers/ids节点相关的变化：

 Broker数量增加或减少的变化

 Broker对应的数据变化

**2、Topic管理**

  新增：监听 /brokers/topics节点相关的变化

  修改：监听 /brokers/topics节点相关的变化

  删除：监听 /admin/delete_topics节点相关的变化

**3、Partation管理**

监听 /admin/reassign_partitions节点相关的变化

监听 /isr_change_notification节点相关的变化

监听 /preferred_replica_election节点相关的变化

数据服务

启动分区状态机和副本状态机

### 2.2.2 启动ZooKeeper

Kafka集群中含有多个服务节点，而分布式系统中经典的主从（Master - Slave）架构就要求从多个服务节点中找一个节点作为集群管理Master，Kafka集群中的这个Master，我们称之为集群控制器Controller

![image-20240428230921798](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428230921798.png) 

**如果此时Controller节点出现故障，它就不能再管理集群功能，那么其他的Slave节点该如何是好呢？**

![image-20240428230943814](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428230943814.png) 

如果从剩余的2个Slave节点中选一个节点出来作为新的集群控制器是不是一个不错的方案，我们将这个选择的过程称之为：选举（elect）。方案是不错，但是问题就在于选哪一个Slave节点呢？不同的软件实现类似的选举功能都会有一些选举算法，而Kafka是依赖于ZooKeeper软件实现Broker节点选举功能。

![image-20240428231009326](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231009326.png) 

**ZooKeeper如何实现Kafka的节点选举呢？这就要说到我们用到ZooKeeper的3个功能：**

1、一个是在ZooKeeper软件中创建节点Node，创建一个Node时，我们会设定这个节点是持久化创建，还是临时创建。所谓的持久化创建，就是Node一旦创建后会一直存在，而临时创建，是根据当前的客户端连接创建的临时节点Node，一旦客户端连接断开，那么这个临时节点Node也会被自动删除，所以这样的节点称之为临时节点。

2、ZooKeeper节点是不允许有重复的,所以多个客户端创建同一个节点，只能有一个创建成功。

3、另外一个是客户端可以在ZooKeeper的节点上增加监听器，用于监听节点的状态变化，一旦监听的节点状态发生变化，那么监听器就会触发响应，实现特定监听功能。

**有了上面的三个知识点，我们这里就介绍一下Kafka是如何利用ZooKeeper实现Controller节点的选举的：**

1) 第一次启动Kafka集群时，会同时启动多个Broker节点，每一个Broker节点就会连接ZooKeeper，并尝试创建一个临时节点 **controller**
2) 因为ZooKeeper中一个节点不允许重复创建，所以多个Broker节点，最终只能有一个Broker节点可以创建成功，那么这个创建成功的Broker节点就会自动作为Kafka集群控制器节点，用于管理整个Kafka集群。
3) 没有选举成功的其他Slave节点会创建Node监听器，用于监听 **controller**节点的状态变化。
4) 一旦Controller节点出现故障或挂掉了，那么对应的ZooKeeper客户端连接就会中断。ZooKeeper中的 **controller** 节点就会自动被删除，而其他的那些Slave节点因为增加了监听器，所以当监听到 **controller** 节点被删除后，就会马上向ZooKeeper发出创建 **controller** 节点的请求，一旦创建成功，那么该Broker就变成了新的Controller节点了。

现在我们能明白启动Kafka集群之前，为什么要先启动ZooKeeper集群了吧。就是因为ZooKeeper可以协助Kafka进行集群管理。

### 2.2.3 启动Kafka

ZooKeeper已经启动好了，那我们现在可以启动多个Kafka Broker节点构建Kafka集群了。构建的过程中，每一个Broker节点就是一个Java进程，而在这个进程中，有很多需要提前准备好，并进行初始化的内部组件对象。

#### 2.2.3.1初始化ZooKeeper

Kafka Broker启动时，首先会先创建ZooKeeper客户端（**KafkaZkClient**），用于和ZooKeeper进行交互。客户端对象创建完成后，会通过该客户端对象向ZooKeeper发送创建Node的请求，注意，这里创建的Node都是持久化Node。

![image-20240428231104886](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231104886.png) 

| **节点**              | **类型** | **说明**                                               |
| --------------------------- | -------------- | ------------------------------------------------------------ |
| /admin/delete_topics        | 持久化节点     | 配置需要删除的topic，因为删除过程中，可能broker下线，或执行失败，那么就需要在broker重新上线后，根据当前节点继续删除操作，一旦topic所有的分区数据全部删除，那么当前节点的数据才会进行清理 |
| /brokers/ids                | 持久化节点     | 服务节点ID标识，只要broker启动，那么就会在当前节点中增加子节点，brokerID不能重复 |
| /brokers/topics             | 持久化节点     | 服务节点中的主题详细信息，包括分区，副本                     |
| /brokers/seqid              | 持久化节点     | seqid主要用于自动生产brokerId                                |
| /config/changes             | 持久化节点     | kafka的元数据发生变化时,会向该节点下创建子节点。并写入对应信息 |
| /config/clients             | 持久化节点     | 客户端配置，默认为空                                         |
| /config/brokers             | 持久化节点     | 服务节点相关配置，默认为空                                   |
| /config/ips                 | 持久化节点     | IP配置，默认为空                                             |
| /config/topics              | 持久化节点     | 主题配置，默认为空                                           |
| /config/users               | 持久化节点     | 用户配置，默认为空                                           |
| /consumers                  | 持久化节点     | 消费者节点，用于记录消费者相关信息                           |
| /isr_change_notification    | 持久化节点     | ISR列表发生变更时候的通知，在kafka当中由于存在ISR列表变更的情况发生,为了保证ISR列表更新的及时性，定义了isr_change_notification这个节点，主要用于通知Controller来及时将ISR列表进行变更。 |
| /latest_producer_id_block   | 持久化节点     | 保存PID块，主要用于能够保证生产者的任意写入请求都能够得到响应。 |
| /log_dir_event_notification | 持久化节点     | 主要用于保存当broker当中某些数据路径出现异常时候,例如磁盘损坏,文件读写失败等异常时候,向ZooKeeper当中增加一个通知序号，Controller节点监听到这个节点的变化之后，就会做出对应的处理操作 |
| /cluster/id                 | 持久化节点     | 主要用于保存kafka集群的唯一id信息，每个kafka集群都会给分配要给唯一id，以及对应的版本号 |

#### 2.2.3.2初始化服务

Kafka Broker中有很多的服务对象，用于实现内部管理和外部通信操作。

![image-20240428231144083](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231144083.png) 

##### 2.2.3.2.1 启动任务调度器

每一个Broker在启动时都会创建内部调度器（**KafkaScheduler**）并启动，用于完成节点内部的工作任务。底层就是Java中的定时任务线程池**ScheduledThreadPoolExecutor**

##### 2.2.3.2.2 创建数据管理器

每一个Broker在启动时都会创建数据管理器（**LogManager**），用于接收到消息后，完成后续的数据创建，查询，清理等处理。

##### 2.2.3.2.3 创建远程数据管理器

每一个Broker在启动时都会创建远程数据管理器（RemoteLogManager），用于和其他Broker节点进行数据状态同步。

##### 2.2.3.2.4 创建副本管理器

每一个Broker在启动时都会创建副本管理器（**ReplicaManager**），用于对主题的副本进行处理。

##### 2.2.3.2.5 创建ZK元数据缓存

每一个Broker在启动时会将ZK的关于Kafka的元数据进行缓存，创建元数据对象（**ZkMetadataCache**）

##### 2.2.3.2.6 创建Broker通信对象

每一个Broker在启动时会创建Broker之间的通道管理器对象（BrokerToControllerChannelManager），用于管理Broker和Controller之间的通信。

##### 2.2.3.2.7 创建网络通信对象

每一个Broker在启动时会创建自己的网络通信对象（**SocketServer**），用于和其他Broker之间的进行通信，其中包含了Java用于NIO通信的Channel、Selector对象。

![image-20240428231200748](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231200748.png) 

##### 2.2.3.2.8 注册Broker节点

Broker启动时，会通过ZK客户端对象向ZK注册当前的Broker 节点ID，注册后创捷的ZK节点为临时节点。如果当前Broker的ZK客户端断开和ZK的连接，注册的节点会被删除。

#### 2.2.3.3启动控制器

控制器（KafkaController）是每一个Broker启动时都会创建的核心对象，用于和ZK之间建立连接并申请自己为整个Kafka集群的Master管理者。如果申请成功，那么会完成管理者的初始化操作，并建立和其他Broker之间的数据通道接收各种事件，进行封装后交给事件管理器，并定义了process方法，用于真正处理各类事件。

![image-20240428231222024](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231222024.png) 

##### 2.2.3.3.1 初始化通道管理器

创建通道管理器（**ControllerChannelManager**），该管理器维护了Controller和集群所有Broker节点之间的网络连接，并向Broker发送控制类请求及接收响应。

##### 2.2.3.3.2 初始化事件管理器

创建事件管理器（**ControllerEventManager**）维护了Controller和集群所有Broker节点之间的网络连接，并向Broker发送控制类请求及接收响应。

##### 2.2.3.3.3 初始化状态管理器

创建状态管理器（**ControllerChangeHandler**）可以监听 /controller 节点的操作，一旦节点创建（ControllerChange），删除（Reelect），数据发生变化（ControllerChange），那么监听后执行相应的处理。

##### 2.2.3.3.4 启动控制器

控制器对象启动后，会向事件管理器发送Startup事件，事件处理线程接收到事件后会通过ZK客户端向ZK申请 /controller 节点，申请成功后，执行当前节点成为Controller的一些列操作。主要是注册各类 ZooKeeper 监听器、删除日志路径变更和 ISR 副本变更通知事件、启动 Controller 通道管理器，以及启动副本状态机和分区状态机。

## 2.3 创建主题

Topic主题是Kafka中消息的逻辑分类，但是这个分类不应该是固定的，而是应该由外部的业务场景进行定义（注意：Kafka中其实是有两个固定的，用于记录消费者偏移量和事务处理的主题），所以Kafka提供了相应的指令和客户端进行主题操作。

### 2.3.1 相关概念

#### 2.3.1.1 主题：Topic

Kafka是分布式消息传输系统，采用的数据传输方式为发布，订阅模式，也就是说由消息的生产者发布消息，消费者订阅消息后获取数据。为了对消费者订阅的消息进行区分，所以对消息在逻辑上进行了分类，这个分类我们称之为主题：**Topic**。消息的生产者必须将消息数据发送到某一个主题，而消费者必须从某一个主题中获取消息，并且消费者可以同时消费一个或多个主题的数据。Kafka集群中可以存放多个主题的消息数据。

为了防止主题的名称和监控指标的名称产生冲突，官方推荐主题的名称中不要同时包含下划线和点。

![image-20240428231241702](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231241702.png)

#### 2.3.1.2 分区：Partition

Kafka消息传输采用发布、订阅模式，所以消息生产者必须将数据发送到一个主题，假如发送给这个主题的数据非常多，那么主题所在broker节点的负载和吞吐量就会受到极大的考验，甚至有可能因为热点问题引起broker节点故障，导致服务不可用。一个好的方案就是将一个主题从物理上分成几块，然后将不同的数据块均匀地分配到不同的broker节点上，这样就可以缓解单节点的负载问题。这个主题的分块我们称之为：分区partition。默认情况下，topic主题创建时分区数量为1，也就是一块分区，可以指定参数--partitions改变。Kafka的分区解决了单一主题topic线性扩展的问题，也解决了负载均衡的问题。

topic主题的每个分区都会用一个编号进行标记，一般是从0开始的连续整数数字。Partition分区是物理上的概念，也就意味着会以数据文件的方式真实存在。每个topic包含一个或多个partition，每个partition都是一个有序的队列。partition中每条消息都会分配一个有序的ID，称之为偏移量：Offset

![image-20240428231306815](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231306815.png) 

#### 2.3.1.3 副本：Replication

分布式系统出现错误是比较常见的，只要保证集群内部依然存在可用的服务节点即可，当然效率会有所降低，不过只要能保证系统可用就可以了。咱们Kafka的topic也存在类似的问题，也就是说，如果一个topic划分了多个分区partition，那么这些分区就会均匀地分布在不同的broker节点上，一旦某一个broker节点出现了问题，那么在这个节点上的分区就会出现问题，那么Topic的数据就不完整了。所以一般情况下，为了防止出现数据丢失的情况，我们会给分区数据设定多个备份，这里的备份，我们称之为：副本Replication。

Kafka支持多副本，使得主题topic可以做到更多容错性，牺牲性能与空间去换取更高的可靠性。

![image-20240428231327573](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231327573.png) 

注意：这里不能将多个备份放置在同一个broker中，因为一旦出现故障，多个副本就都不能用了，那么副本的意义就没有了。

#### 2.3.1.4 副本类型：Leader & Follower

假设我们有一份文件，一般情况下，我们对副本的理解应该是有一个正式的完整文件，然后这个文件的备份，我们称之为副本。但是在Kafka中，不是这样的，所有的文件都称之为副本，只不过会选择其中的一个文件作为主文件，称之为：Leader(主导)副本，其他的文件作为备份文件，称之为：Follower（追随）副本。在Kafka中，这里的文件就是分区，每一个分区都可以存在1个或多个副本，只有Leader副本才能进行数据的读写，Follower副本只做备份使用。

![image-20240428231345209](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231345209.png)



#### 2.3.1.5 日志：Log

Kafka最开始的应用场景就是日志场景或MQ场景，更多的扮演着一个日志传输和存储系统，这是Kafka立家之本。所以Kafka接收到的消息数据最终都是存储在log日志文件中的，底层存储数据的文件的扩展名就是log。

主题创建后，会创建对应的分区数据Log日志。并打开文件连接通道，随时准备写入数据。

### 2.3.2 创建第一个主题

创建主题Topic的方式有很多种：命令行，工具，客户端API，自动创建。在server.properties文件中配置参数auto.create.topics.enable=true时，如果访问的主题不存在，那么Kafka就会自动创建主题，这个操作不在我们的讨论范围内。由于我们学习的重点在于学习原理和基础概念，所以这里我们选择比较基础的命令行方式即可。

我们首先创建的主题，仅仅指明主题的名称即可，其他参数暂时无需设定。

#### 2.3.2.1 执行指令

```
bin/kafka-topics.sh --bootstrap-server kafka-broker1:9092 --create --topic first-topic
```
#### 2.3.2.2 ZooKeeper节点变化

指令执行后，当前Kafka会增加一个主题，因为指令中没有配置分区和副本参数，所以当前主题分区数量为默认值1，编号为0，副本为1，编号为所在broker的ID值。为了方便集群的管理，创建topic时，会同时在ZK中增加子节点，记录主题相关配置信息：

/config/topics节点中会增加first-topic节点。

![image-20240428231432153](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231432153.png) 

/brokers/topics节点中会增加first-topic节点以及相应的子节点。

![image-20240428231447776](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231447776.png) 

| **节点**                         | **节点类型** | **数据名称**                                           | **数据值** | **说明**                                          |
| -------------------------------------- | ------------------ | ------------------------------------------------------------ | ---------------- | ------------------------------------------------------- |
| /topics/first-topic                    | 持久类型           | removing_replicas                                            | 无               |                                                         |
| partitions                             | {"0":[3]}          | 分区配置                                                     |                  |                                                         |
| topic_id                               | 随机字符串         |                                                              |                  |                                                         |
| adding_replicas                        | 无                 |                                                              |                  |                                                         |
| version                                | 3                  |                                                              |                  |                                                         |
| /topics/first-topic/partitions         | 持久类型           |                                                              |                  | 主题分区节点，每个主题都应该设置分区，保存在该节点      |
| /topics/first-topic/partitions/0       | 持久类型           |                                                              |                  | 主题分区副本节点，因为当前主题只有一个分区，所以编号为0 |
| /topics/first-topic/partitions/0/state | 持久类型           | controller_epoch                                             | 7                | 主题分区副本状态节点                                    |
| leader                                 | 3                  | Leader副本所在的broker Id                                    |                  |                                                         |
| version                                | 1                  |                                                              |                  |                                                         |
| leader_epoch                           | 0                  |                                                              |                  |                                                         |
| isr                                    | [3]                | 副本同步列表，因为当前只有一个副本，所以副本中只有一个副本编号 |                  |                                                         |

#### 2.3.2.3 数据存储位置

主题创建后，需要找到一个用于存储分区数据的位置，根据上面ZooKeeper存储的节点配置信息可以知道，当前主题的分区数量为1，副本数量为1，那么数据存储的位置就是副本所在的broker节点，从当前数据来看，数据存储在我们的第三台broker上。
```
cd /opt/module/kafka/datas
```
![image-20240428231504895](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231504895.png) 
```
cd first-topic-0
```

![image-20240428231523049](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231523049.png) 

路径中的00000000000000000000.log文件就是真正存储消息数据的文件，文件名称中的0表示当前文件的起始偏移量为0，index文件和timeindex文件都是数据索引文件，用于快速定位数据。只不过index文件采用偏移量的方式进行定位，而timeindex是采用时间戳的方式。

### 2.3.3 创建第二个主题

接下来我们创建第二个主题，不过创建时，我们需要设定分区参数 --partitions，参数值为3，表示创建3个分区

#### 2.3.3.1 执行指令

![image-20240428231533708](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231533708.png)
```
bin/kafka-topics.sh --bootstrap-server kafka-broker1:9092 --create --topic second-topic --partitions 3
```
#### 2.3.3.2 ZooKeeper节点变化

指令执行后，当前Kafka会增加一个主题，因为指令中指定了分区数量（--partitions 3），所以当前主题分区数量为3，编号为[0、1、2]，副本为1，编号为所在broker的ID值。为了方便集群的管理，创建Topic时，会同时在ZK中增加子节点，记录主题相关配置信息：
/config/topics节点中会增加second-topic节点。
![image-20240428231551653](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231551653.png) 

 /brokers/topics节点中会增加second-topic节点以及相应的子节点。

![image-20240428231605723](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231605723.png)

 

| **节点**                          | **节点类型**        | **数据名称**                                           | **数据值** | **说明**                                               |
| --------------------------------------- | ------------------------- | ------------------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| /topics/second-topic                    | 持久类型                  | removing_replicas                                            | 无               |                                                              |
| partitions                              | {"2":[3],"1":[2],"0":[1]} | 分区配置                                                     |                  |                                                              |
| topic_id                                | 随机字符串                |                                                              |                  |                                                              |
| adding_replicas                         | 无                        |                                                              |                  |                                                              |
| version                                 | 3                         |                                                              |                  |                                                              |
| /topics/second-topic/partitions         | 持久类型                  |                                                              |                  | 主题分区节点，每个主题都应该设置分区，保存在该节点           |
| /topics/second-topic/partitions/0       | 持久类型                  |                                                              |                  | 主题分区副本节点，因为当前主题有3个分区，第一个分区编号为0   |
| /topics/second-topic/partitions/0/state | 持久类型                  | controller_epoch                                             | 7                | 主题分区副本状态节点                                         |
| leader                                  | 1                         | Leader副本所在的broker Id                                    |                  |                                                              |
| version                                 | 1                         |                                                              |                  |                                                              |
| leader_epoch                            | 0                         |                                                              |                  |                                                              |
| isr                                     | [1]                       | 副本同步列表，因为当前只有一个副本，所以副本中只有一个副本编号 |                  |                                                              |
| /topics/second-topic/partitions/1       | 持久类型                  |                                                              |                  | 主题分区副本节点，因为当前主题有3个分区，当前为第2个分区，所以编号为1 |
| /topics/second-topic/partitions/1/state | 持久类型                  | controller_epoch                                             | 7                | 主题分区副本状态节点                                         |
| leader                                  | 2                         | Leader副本所在的broker Id                                    |                  |                                                              |
| version                                 | 1                         |                                                              |                  |                                                              |
| leader_epoch                            | 0                         |                                                              |                  |                                                              |
| isr                                     | [2]                       | 副本同步列表，因为当前只有一个副本，所以副本中只有一个副本编号 |                  |                                                              |
| /topics/second-topic/partitions/2       | 持久类型                  |                                                              |                  | 主题分区副本节点，因为当前主题有3个分区，当前为第3个分区，所以编号为2 |
| /topics/second-topic/partitions/2/state | 持久类型                  | controller_epoch                                             | 7                | 主题分区副本状态节点                                         |
| leader                                  | 3                         | Leader副本所在的broker Id                                    |                  |                                                              |
| version                                 | 1                         |                                                              |                  |                                                              |
| leader_epoch                            | 0                         |                                                              |                  |                                                              |
| isr                                     | [3]                       | 副本同步列表，因为当前只有一个副本，所以副本中只有一个副本编号 |                  |                                                              |

#### 2.3.3.3 数据存储位置

主题创建后，需要找到一个用于存储分区数据的位置，根据上面ZooKeeper存储的节点配置信息可以知道，当前主题的分区数量为3，副本数量为1，那么数据存储的位置就是每个分区Leader副本所在的broker节点。
```
 cd /opt/module/kafka/datas
```

![image-20240428231629017](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231629017.png) 
```
cd second-topic-0
ll
```
![image-20240428231736243](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231736243.png) 
```
cd /opt/module/kafka/datas
ll
```
![image-20240428231751032](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231751032.png) 
```
cd second-topic-1
ll
```
![image-20240428231806602](https://raw.githubusercontent.com/PeipengWang/picture/master/image-20240428231806602.png)


### 2.3.4 创建第三个主题

接下来我们创建第三个主题，不过创建时，我们需要设定副本参数 --replication-factor，参数值为3，表示每个分区创建3个副本。

#### 2.3.4.1 执行指令
```
bin/kafka-topics.sh --bootstrap-server kafka-broker1:9092 --create --topic third-topic --partitions 3 --replication-factor 3
```
#### 2.3.4.2 ZooKeeper节点变化

指令执行后，当前Kafka会增加一个主题，因为指令中指定了分区数量和副本数量（--replication-factor 3），所以当前主题分区数量为3，编号为[0、1、2]，副本为3，编号为[1、2、3]。为了方便集群的管理，创建Topic时，会同时在ZK中增加子节点，记录主题相关配置信息：

Ø /config/topics节点中会增加third-topic节点。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps39.jpg) 

Ø /brokers/topics节点中会增加third-topic节点以及相应的子节点。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps40.jpg) 

| **节点**                         | **节点类型**                    | **数据名称**                                           | **数据值** | **说明**                                               |
| -------------------------------------- | ------------------------------------- | ------------------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| /topics/third-topic                    | 持久类型                              | removing_replicas                                            | 无               |                                                              |
| partitions                             | {"2":[1,2,3],"1":[3,1,2],"0":[2,3,1]} | 分区配置                                                     |                  |                                                              |
| topic_id                               | 随机字符串                            |                                                              |                  |                                                              |
| adding_replicas                        | 无                                    |                                                              |                  |                                                              |
| version                                | 3                                     |                                                              |                  |                                                              |
| /topics/third-topic/partitions         | 持久类型                              |                                                              |                  | 主题分区节点，每个主题都应该设置分区，保存在该节点           |
| /topics/third-topic/partitions/0       | 持久类型                              |                                                              |                  | 主题分区副本节点，因为当前主题有3个分区，第一个分区编号为0   |
| /topics/third-topic/partitions/0/state | 持久类型                              | controller_epoch                                             | 7                | 主题分区副本状态节点                                         |
| leader                                 | 2                                     | Leader副本所在的broker Id                                    |                  |                                                              |
| version                                | 1                                     |                                                              |                  |                                                              |
| leader_epoch                           | 0                                     |                                                              |                  |                                                              |
| isr                                    | [2,3,1]                               | 副本同步列表，因为当前有3个副本，所以列表中的第一个副本就是Leader副本，其他副本均为follower副本 |                  |                                                              |
| /topics/third-topic/partitions/1       | 持久类型                              |                                                              |                  | 主题分区副本节点，因为当前主题有3个分区，当前为第2个分区，所以编号为1 |
| /topics/third-topic/partitions/1/state | 持久类型                              | controller_epoch                                             | 7                | 主题分区副本状态节点                                         |
| leader                                 | 3                                     | Leader副本所在的broker Id                                    |                  |                                                              |
| version                                | 1                                     |                                                              |                  |                                                              |
| leader_epoch                           | 0                                     |                                                              |                  |                                                              |
| isr                                    | [3,1,2]                               | 副本同步列表，因为当前有3个副本，所以列表中的第一个副本就是Leader副本，其他副本均为follower副本 |                  |                                                              |
| /topics/third-topic/partitions/2       | 持久类型                              |                                                              |                  | 主题分区副本节点，因为当前主题有3个分区，当前为第3个分区，所以编号为2 |
| /topics/third-topic/partitions/2/state | 持久类型                              | controller_epoch                                             | 7                | 主题分区副本状态节点                                         |
| leader                                 | 1                                     | Leader副本所在的broker Id                                    |                  |                                                              |
| version                                | 1                                     |                                                              |                  |                                                              |
| leader_epoch                           | 0                                     |                                                              |                  |                                                              |
| isr                                    | [1,2,3]                               | 副本同步列表，因为当前有3个副本，所以列表中的第一个副本就是Leader副本，其他副本均为follower副本 |                  |                                                              |

#### 2.3.4.3 数据存储位置

主题创建后，需要找到一个用于存储分区数据的位置，根据上面ZooKeeper存储的节点配置信息可以知道，当前主题的分区数量为3，副本数量为3，那么数据存储的位置就是每个分区副本所在的broker节点。

```
cd /opt/module/kafka/datas
```

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps41.jpg) 

```
cd third-topic-2
```

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps42.jpg) 

```
 cd /opt/module/kafka/datas
```

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps43.jpg) 

```
 cd third-topic-0
```

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps44.jpg) 

```
cd /opt/module/kafka/datas
```

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps45.jpg) 

```
 cd third-topic-1
```

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps46.jpg) 

### 2.3.5 创建主题流程

Kafka中主题、分区以及副本的概念都和数据存储相关，所以是非常重要的。前面咱们演示了一下创建主题的具体操作和现象，那么接下来，我们就通过图解来了解一下Kafka是如何创建主题，并进行分区副本分配的。

#### 2.3.5.1 命令行提交创建指令

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps47.jpg) 

1) 通过命令行提交指令，指令中会包含操作类型（**--create**）、topic的名称（**--topic**）、主题分区数量（**--partitions**）、主题分区副本数量（**--replication-facotr**）、副本分配策略（**--replica-assignment**）等参数。
2) 指令会提交到客户端进行处理，客户端获取指令后，会首先对指令参数进行校验。

a. 操作类型取值：create、list、alter、describe、delete，只能存在一个。

b. 分区数量为大于1的整数。

c. 主题是否已经存在

d. 分区副本数量大于1且小于Short.MaxValue，一般取值小于等于Broker数量。

3) 将参数封装主题对象（NewTopic）。
4) 创建通信对象，设定请求标记（CREATE_TOPICS），查找Controller，通过通信对象向Controller发起创建主题的网络请求。

#### 2.3.5.2 Controller接收创建主题请求

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps48.jpg) 

(1) Controller节点接收到网络请求（Acceptor），并将请求数据封装成请求对象放置在队列（requestQueue）中。

(2) 请求控制器（KafkaRequestHandler）周期性从队列中获取请求对象（BaseRequest）。

(3) 将请求对象转发给请求处理器（KafkaApis），根据请求对象的类型调用创建主题的方法。

#### 2.3.5.3 创建主题

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps49.jpg) 

(1) 请求处理器（KafkaApis）校验主题参数。

如果分区数量没有设置，那么会采用Kafka启动时加载的配置项：**num.partitions**（默认值为1）

如果副本数量没有设置，那么会采用Kafka启动时记载的配置项：

**default.replication.factor**（默认值为1）

(2) 在创建主题时，如果使用了replica-assignment参数，那么就按照指定的方案来进行分区副本的创建；如果没有指定replica-assignment参数，那么就按照Kafka内部逻辑来分配，内部逻辑按照机架信息分为两种策略：【未指定机架信息】和【指定机架信息】。**当前课程中采用的是【未指定机架信息】副本分配策略：**

分区起始索引设置0

轮询所有分区，计算每一个分区的所有副本位置：

副本起始索引 = （分区编号 + 随机值） %  BrokerID列表长度。

其他副本索引 =  随机值（基本算法为使用随机值执行多次模运算）

**实例：**

```
假设 当前分区编号 : 0
BrokerID列表 :【1，2，3，4】
副本数量 : 4
随机值（BrokerID列表长度）: 2
副本分配间隔随机值（BrokerID列表长度）: 2
第一个副本索引：（分区编号 + 随机值）% BrokerID列表长度 =（0 + 2）% 4 = 2
第一个副本所在BrokerID : 3
第二个副本索引（第一个副本索引 + （1 +（副本分配间隔 + 0）% （BrokerID列表长度 - 1））） % BrokerID列表长度 = （2 +（1+（2+0）%3））% 4 = 1
第二个副本所在BrokerID：2
 第三个副本索引：（第一个副本索引 + （1 +（副本分配间隔 + 1）% （BrokerID列表长度 - 1））） % BrokerID列表长度 = （2 +（1+（2+1）%3））% 4 = 3
 第三个副本所在BrokerID：4
 第四个副本索引：（第一个副本索引 + （1 +（副本分配间隔 + 2）% （BrokerID列表长度 - 1））） % BrokerID列表长度 = （2 +（1+（2+2）%3））% 4 = 0\
第四个副本所在BrokerID：1
 最终分区0的副本所在的Broker节点列表为【3，2，4，1】
```

 **其他分区采用同样算法**

通过索引位置获取副本节点ID

保存分区以及对应的副本ID列表。

(3) 通过ZK客户端在ZK端创建节点：

在 /config/topics节点下，增加当前主题节点，节点类型为持久类型。

在 /brokers/topics节点下，增加当前主题及相关节点，节点类型为持久类型。

(4) Controller节点启动后，会在/brokers/topics节点增加监听器，一旦节点发生变化，会触发相应的功能：

获取需要新增的主题信息

更新当前Controller节点保存的主题状态数据

更新分区状态机的状态为：NewPartition

更新副本状态机的状态：NewReplica

更新分区状态机的状态为：OnlinePartition，从正常的副本列表中的获取第一个作为分区的**Leader副本**，所有的副本作为分区的同步副本列表，我们称之为**ISR( In-Sync Replica)。**在ZK路径brokers/topics/主题名**上增加分区节点**partitions，**及状态**state**节点。

更新副本状态机的状态：OnlineReplica

(5) Controller节点向主题的各个分区副本所属Broker节点发送LeaderAndIsrRequest请求，向所有的Broker发送UPDATE_METADATA请求，更新自身的缓存。

Controller向分区所属的Broker发送请求

Broker节点接收到请求后，根据分区状态信息，设定当前的副本为Leader或Follower，并创建底层的数据存储文件目录和空的数据文件。

文件目录名：主题名 + 分区编号

| **文件名**           | **说明**               |
| -------------------------- | ---------------------------- |
| 0000000000000000.log       | 数据文件，用于存储传输的小心 |
| 0000000000000000.index     | 索引文件，用于定位数据       |
| 0000000000000000.timeindex | 时间索引文件，用于定位数据   |

## **2.4** **生产消息**

Topic主题已经创建好了，接下来我们就可以向该主题生产消息了，这里我们采用Java代码通过Kafka Producer API的方式生产数据。

### **2.4.1** **生产消息的基本步骤**

**（一）创建Map类型的配置对象，根据场景增加相应的配置属性：**

| **参数名***                      | **参数作用**                                           | **类型** | **默认值** | **推荐值**                            |
| ------------------------------------- | ------------------------------------------------------------ | -------------- | ---------------- | ------------------------------------------- |
| bootstrap.servers                     | 集群地址，格式为：brokerIP1:端口号,brokerIP2:端口号          | 必须           |                  |                                             |
| key.serializer                        | 对生产数据Key进行序列化的类完整名称                          | 必须           |                  | Kafka提供的字符串序列化类：StringSerializer |
| value.serializer                      | 对生产数据Value进行序列化的类完整名称                        | 必须           |                  | Kafka提供的字符串序列化类：StringSerializer |
| interceptor.classes                   | 拦截器类名，多个用逗号隔开                                   | 可选           |                  |                                             |
| batch.size                            | 数据批次字节大小。此大小会和数据最大估计值进行比较，取大值。估值=61+21+（keySize+1+valueSize+1+1） | 可选           | 16K              |                                             |
| retries                               | 重试次数                                                     | 可选           | 整型最大值       | 0或整型最大值                               |
| request.timeout.ms                    | 请求超时时间                                                 | 可选           | 30s              |                                             |
| linger.ms                             | 数据批次在缓冲区中停留时间                                   | 可选           |                  |                                             |
| acks                                  | 请求应答类型：all(-1), 0, 1                                  | 可选           | all(-1)          | 根据数据场景进行设置                        |
| retry.backoff.ms                      | 两次重试之间的时间间隔                                       | 可选           | 100ms            |                                             |
| buffer.memory                         | 数据收集器缓冲区内存大小                                     | 可选           | 32M              | 64M                                         |
| max.in.flight.requests.per.connection | 每个节点连接的最大同时处理请求的数量                         | 可选           | 5                | 小于等于5                                   |
| enable.idempotence                    | 幂等性，                                                     | 可选           | true             | 根据数据场景进行设置                        |
| partitioner.ignore.keys               | 是否放弃使用数据key选择分区                                  | 可选           | false            |                                             |
| partitioner.class                     | 分区器类名                                                   | 可选           | null             |                                             |

**（二）创建待发送数据**

在kafka中传递的数据我们称之为消息（message）或记录(record)，所以Kafka发送数据前，需要将待发送的数据封装为指定的数据模型：

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps50.jpg) 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps51.jpg) 

相关属性必须在构建数据模型时指定，其中主题和value的值是必须要传递的。如果配置中开启了自动创建主题，那么Topic主题可以不存在。value就是我们需要真正传递的数据了，而Key可以用于数据的分区定位。

**（三）创建生产者对象，发送生产的数据：**

根据前面提供的配置信息创建生产者对象，通过这个生产者对象向Kafka服务器节点发送数据，而具体的发送是由生产者对象创建时，内部构建的多个组件实现的，多个组件的关系有点类似于生产者消费者模式。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps52.jpg) 

(1) 数据生产者（**KafkaProducer**）：生产者对象，用于对我们的数据进行必要的转换和处理，将处理后的数据放入到数据收集器中，类似于生产者消费者模式下的生产者。这里我们简单介绍一下内部的数据转换处理：

 如果配置拦截器栈（interceptor.classes），那么将数据进行拦截处理。某一个拦截器出现异常并不会影响后续的拦截器处理。

因为发送的数据为KV数据，所以需要根据配置信息中的序列化对象对数据中Key和Value分别进行序列化处理。

计算数据所发送的分区位置。

将数据追加到数据收集器中。

(2) 数据收集器（**RecordAccumulator**）：用于收集，转换我们产生的数据，类似于生产者消费者模式下的缓冲区。为了优化数据的传输，Kafka并不是生产一条数据就向Broker发送一条数据，而是通过合并单条消息，进行批量（批次）发送，提高吞吐量，减少带宽消耗。

默认情况下，一个发送批次的数据容量为16K，这个可以通过参数batch.size进行改善。

批次是和分区进行绑定的。也就是说发往同一个分区的数据会进行合并，形成一个批次。

如果当前批次能容纳数据，那么直接将数据追加到批次中即可，如果不能容纳数据，那么会产生新的批次放入到当前分区的批次队列中，这个队列使用的是Java的双端队列Deque。旧的批次关闭不再接收新的数据，等待发送

(3) 数据发送器（**Sender**）：线程对象，用于从收集器对象中获取数据，向服务节点发送。类似于生产者消费者模式下的消费者。因为是线程对象，所以启动后会不断轮询获取数据收集器中已经关闭的批次数据。对批次进行整合后再发送到Broker节点中

因为数据真正发送的地方是Broker节点，不是分区。所以需要将从数据收集器中收集到的批次数据按照可用Broker节点重新组合成List集合。

将组合后的<节点，List<批次>>的数据封装成客户端请求（请求键为：Produce）发送到网络客户端对象的缓冲区，由网络客户端对象通过网络发送给Broker节点。

Broker节点获取客户端请求，并根据请求键进行后续的数据处理：向分区中增加数据。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps53.jpg) 

### 2.4.2 生产消息的基本代码 

```
// TODO 配置属性集合

Map<String, Object> configMap = new HashMap<>();

// TODO 配置属性：Kafka服务器集群地址

configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

// TODO 配置属性：Kafka生产的数据为KV对，所以在生产数据进行传输前需要分别对K,V进行对应的序列化操作

configMap.put(

 ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,

  "org.apache.kafka.common.serialization.StringSerializer");

configMap.put(

 ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,

 "org.apache.kafka.common.serialization.StringSerializer");

// TODO 创建Kafka生产者对象，建立Kafka连接

//    构造对象时，需要传递配置参数

KafkaProducer<String, String> producer = new KafkaProducer<>(configMap);

// TODO 准备数据,定义泛型

//    构造对象时需要传递 【Topic主题名称】，【Key】，【Value】三个参数

ProducerRecord<String, String> record = new ProducerRecord<String, String>(

 "test", "key1", "value1"

);

// TODO 生产（发送）数据

producer.send(record);

// TODO 关闭生产者连接

producer.close();
```

### 2.4.3 发送消息

#### 2.4.3.1拦截器

生产者API在数据准备好发送给Kafka服务器之前，允许我们对生产的数据进行统一的处理，比如校验，整合数据等等。这些处理我们是可以通过Kafka提供的拦截器完成。因为拦截器不是生产者必须配置的功能，所以大家可以根据实际的情况自行选择使用。

但是要注意，这里的拦截器是可以配置多个的。执行时，会按照声明顺序执行完一个后，再执行下一个。并且某一个拦截器如果出现异常，只会跳出当前拦截器逻辑，并不会影响后续拦截器的处理。所以开发时，需要将拦截器的这种处理方法考虑进去。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps55.jpg) 

接下来，我们来演示一下拦截器的操作：

##### 2.4.3.1.1 增加拦截器类

(1) 实现生产者拦截器接口ProducerInterceptor
```
package com.atguigu.test;

import org.apache.kafka.clients.producer.ProducerInterceptor;

import org.apache.kafka.clients.producer.ProducerRecord;

import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Map;

/**

 \* TODO 自定义数据拦截器

 \*    1. 实现Kafka提供的生产者接口ProducerInterceptor

 \*    2. 定义数据泛型 <K, V>

 \*    3. 重写方法

 \*     onSend

 \*     onAcknowledgement

 \*     close

 \*     configure

 */

public class KafkaInterceptorMock implements ProducerInterceptor<String, String> {

  @Override

  public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
  return record;

  }

  @Override	

  public void onAcknowledgement(RecordMetadata metadata, Exception exception) {

  }

  @Override

  public void close() {

  }

  @Override

  public void configure(Map<String, ?> configs) {

  }

}
```
(2) 实现接口中的方法，根据业务功能重写具体的方法

| **方法名**  | **作用**                                               |
| ----------------- | ------------------------------------------------------------ |
| onSend            | 数据发送前，会执行此方法，进行数据发送前的预处理             |
| onAcknowledgement | 数据发送后，获取应答时，会执行此方法                         |
| close             | 生产者关闭时，会执行此方法，完成一些资源回收和释放的操作     |
| configure         | 创建生产者对象的时候，会执行此方法，可以根据场景对生产者对象的配置进行统一修改或转换。 |

##### 2.4.3.1.2 配置拦截器
```
package com.atguigu.test;

import org.apache.kafka.clients.producer.*;

import org.apache.kafka.common.serialization.StringSerializer;

import java.util.HashMap;

import java.util.Map;

import java.util.concurrent.Future;

public class ProducerInterceptorTest {

  public static void main(String[] args) {

   Map<String, Object> configMap = new HashMap<>();

   configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

    configMap.put( ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

    configMap.put( ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

 

  configMap.put( ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, KafkaInterceptorMock.class.getName());

 

   KafkaProducer<String, String> producer = null;

   try {

    producer = new KafkaProducer<>(configMap);

     for ( int i = 0; i < 1; i++ ) {

       ProducerRecord<String, String> record = new ProducerRecord<String, String>("test", "key" + i, "value" + i);

       final Future<RecordMetadata> send = producer.send(record);

    }

   } catch ( Exception e ) {
    e.printStackTrace();
   } finally {

     if ( producer != null ) {

       producer.close();

     }

   }

 

  }

}
```
#### 2.4.3.2 回调方法

Kafka发送数据时，可以同时传递回调对象（Callback）用于对数据的发送结果进行对应处理，具体代码实现采用匿名类或Lambda表达式都可以。

```
package com.atguigu.kafka.test;

import org.apache.kafka.clients.producer.*;

import java.util.HashMap;

import java.util.Map;

public class KafkaProducerASynTest {

  public static void main(String[] args) {

  Map<String, Object> configMap = new HashMap<>();

    configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   configMap.put(

       ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
       "org.apache.kafka.common.serialization.StringSerializer");
   configMap.put(

       ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,

       "org.apache.kafka.common.serialization.StringSerializer");

   KafkaProducer<String, String> producer = new KafkaProducer<>(configMap);

   //  循环生产数据

   for ( int i = 0; i < 1; i++ ) {

     //  创建数据

     ProducerRecord<String, String> record = new ProducerRecord<String, String>("test", "key" + i, "value" + i);

     //  发送数据

     producer.send(record, new Callback() {

       //  回调对象

       public void onCompletion(RecordMetadata recordMetadata, Exception e) {

         //  当数据发送成功后，会回调此方法

         System.out.println("数据发送成功：" + recordMetadata.timestamp());

       }

     });

   }

   producer.close();

  }

}
```
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps57.jpg) 

#### 2.4.3.3异步发送

Kafka发送数据时，底层的实现类似于生产者消费者模式。对应的，底层会由主线程代码作为生产者向缓冲区中放数据，而数据发送线程会从缓冲区中获取数据进行发送。Broker接收到数据后进行后续处理。

如果Kafka通过主线程代码将一条数据放入到缓冲区后，无需等待数据的后续发送过程，就直接发送一下条数据的场合，我们就称之为异步发送。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps58.jpg)  

```

package com.atguigu.kafka.test;

 

import org.apache.kafka.clients.producer.*;

 

import java.util.HashMap;

import java.util.Map;


public class KafkaProducerASynTest {

  public static void main(String[] args) {

   Map<String, Object> configMap = new HashMap<>();

  configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   configMap.put(

      ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,

      "org.apache.kafka.common.serialization.StringSerializer");

   configMap.put(

       ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,

       "org.apache.kafka.common.serialization.StringSerializer");

   KafkaProducer<String, String> producer = new KafkaProducer<>(configMap);
    //  循环生产数据

   for ( int i = 0; i < 10; i++ ) {

     //  创建数据

     ProducerRecord<String, String> record = new ProducerRecord<String, String>("test", "key" + i, "value" + i);

    //  发送数据

    producer.send(record, new Callback() {

      //  回调对象

      public void onCompletion(RecordMetadata recordMetadata, Exception e) {

        //  当数据发送成功后，会回调此方法

        System.out.println("数据发送成功：" + recordMetadata.timestamp());

      }

    });

    //  发送当前数据

    System.out.println("发送数据");

  }

   producer.close();

  }

}
```
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps60.jpg)![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps61.jpg) 

#### 2.4.3.4同步发送

Kafka发送数据时，底层的实现类似于生产者消费者模式。对应的，底层会由主线程代码作为生产者向缓冲区中放数据，而数据发送线程会从缓冲区中获取数据进行发送。Broker接收到数据后进行后续处理。

如果Kafka通过主线程代码将一条数据放入到缓冲区后，需等待数据的后续发送操作的应答状态，才能发送一下条数据的场合，我们就称之为同步发送。所以这里的所谓同步，就是生产数据的线程需要等待发送线程的应答（响应）结果。

代码实现上，采用的是JDK1.5增加的JUC并发编程的Future接口的get方法实现。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps62.jpg)  

```

package com.atguigu.kafka.test;


import org.apache.kafka.clients.producer.*;
import java.util.HashMap;

import java.util.Map;
public class KafkaProducerASynTest {

  public static void main(String[] args) throws Exception {

   Map<String, Object> configMap = new HashMap<>();

   configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   configMap.put(

       ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,

       "org.apache.kafka.common.serialization.StringSerializer");

   configMap.put(

       ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,

       "org.apache.kafka.common.serialization.StringSerializer");

   KafkaProducer<String, String> producer = new KafkaProducer<>(configMap);

   //  循环生产数据

   for ( int i = 0; i < 10; i++ ) {

     //  创建数据

     ProducerRecord<String, String> record = new ProducerRecord<String, String>("test", "key" + i, "value" + i);

    //  发送数据

    producer.send(record, new Callback() {

       // 回调对象

       public void onCompletion(RecordMetadata recordMetadata, Exception e) {

         // 当数据发送成功后，会回调此方法

        System.out.println("数据发送成功：" + recordMetadata.timestamp());
       }

     }).get(); //这个get就是堵塞在这里，直到得到响应
     //  发送当前数据

     System.out.println("发送数据");

   }

   producer.close();

  }

}
```
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps64.jpg) 

### 2.4.4 消息分区

#### 2.4.4.1指定分区

Kafka中Topic是对数据逻辑上的分类，而Partition才是数据真正存储的物理位置。所以在生产数据时，如果只是指定Topic的名称，其实Kafka是不知道将数据发送到哪一个Broker节点的。我们可以在构建数据传递Topic参数的同时，也可以指定数据存储的分区编号。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps65.jpg) 
```
for ( int i = 0; i < 1; i++ ) {

  ProducerRecord<String, String> record = new ProducerRecord<String, String>("test", 0, "key" + i, "value" + i);

  final Future<RecordMetadata> send = producer.send(record, new Callback() {

   public void onCompletion(RecordMetadata recordMetadata, Exception e) {

    if ( e != null ) {
       e.printStackTrace();

     } else {

     System.out.println("数据发送成功：" + record.key() + "," + record.value());

     }

   }

  });

}
```
#### 2.4.4.2未指定分

指定分区传递数据是没有任何问题的。Kafka会进行基本简单的校验，比如是否为空，是否小于0之类的，但是你的分区是否存在就无法判断了，所以需要从Kafka中获取集群元数据信息，此时会因为长时间获取不到元数据信息而出现超时异常。所以如果不能确定分区编号范围的情况，不指定分区还是一个不错的选择。

如果不指定分区，Kafka会根据集群元数据中的主题分区来通过算法来计算分区编号并设定：

(1) 如果指定了分区，直接使用

(2) 如果指定了自己的分区器，通过分区器计算分区编号，如果有效，直接使用

(3) 如果指定了数据Key，且使用Key选择分区的场合，采用murmur2非加密散列算法（类似于hash）计算数据Key序列化后的值的散列值，然后对主题分区数量模运算取余，最后的结果就是分区编号

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps66.jpg) 

(4) 如果未指定数据Key，或不使用Key选择分区，那么Kafka会采用优化后的粘性分区策略进行分区选择：

没有分区数据加载状态信息时，会从分区列表中随机选择一个分区。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps67.jpg) 

如果存在分区数据加载状态信息时，根据分区数据队列加载状态，通过随机数获取一个权重值

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps68.jpg) 

根据这个权重值在队列加载状态中进行二分查找法，查找权重值的索引值

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps69.jpg) 

将这个索引值加1就是当前设定的分区。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps70.jpg) 

增加数据后，会根据当前粘性分区中生产的数据量进行判断，是不是需要切换其他的分区。判断地标准就是大于等于批次大小（16K）的2倍，或大于一个批次大小（16K）且需要切换。如果满足条件，下一条数据就会放置到其他分区。

#### 2.4.4.3分区器

在某些场合中，指定的数据我们是需要根据自身的业务逻辑发往指定的分区的。所以需要自己定义分区编号规则，而不是采用Kafka自动设置就显得尤其必要了。Kafka早期版本中提供了两个分区器，不过在当前kafka版本中已经不推荐使用了。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps71.jpg) 

接下来我们就说一下当前版本Kafka中如何定义我们自己的分区规则：分区器

##### 2.4.4.3.1 增加分区器类（自定义分区器）

首先我们需要创建一个类，然后实现Kafka提供的分区类接口Partitioner，接下来重写方法。这里我们只关注partition方法即可，因为此方法的返回结果就是需要的分区编号。
```
package com.atguigu.test;

 

import org.apache.kafka.clients.producer.Partitioner;

import org.apache.kafka.common.Cluster;

 

import java.util.Map;

 

/**

 \* TODO 自定义分区器实现步骤：

 \*    1. 实现Partitioner接口

 \*    2. 重写方法

 \*     partition : 返回分区编号，从0开始

 \*     close

 \*     configure

 */

public class KafkaPartitionerMock implements Partitioner {

  /**

   \* 分区算法 - 根据业务自行定义即可

   \* @param topic The topic name

   \* @param key The key to partition on (or null if no key)

   \* @param keyBytes The serialized key to partition on( or null if no key)

   \* @param value The value to partition on or null

   \* @param valueBytes The serialized value to partition on or null

   \* @param cluster The current cluster metadata

   \* @return 分区编号，从0开始

   */

  @Override

  public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {

   return 0;

  }

 

  @Override

  public void close() {

 

  }

 

  @Override

  public void configure(Map<String, ?> configs) {

 

  }

}

##### **2.4.4.3.2 配置分区器**

package com.atguigu.test;


import org.apache.kafka.clients.producer.*;

import org.apache.kafka.common.serialization.StringSerializer;

import java.util.HashMap;

import java.util.Map;

import java.util.concurrent.Future;


public class ProducerPartitionTest {

  public static void main(String[] args) {

   Map<String, Object> configMap = new HashMap<>();

    configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   configMap.put( ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

    configMap.put( ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

   configMap.put( ProducerConfig.PARTITIONER_CLASS_CONFIG, KafkaPartitionerMock.class.getName());
//自定义分区器

   KafkaProducer<String, String> producer = null;
   try {
     producer = new KafkaProducer<>(configMap);

     for ( int i = 0; i < 1; i++ ) {
       ProducerRecord<String, String> record = new ProducerRecord<String, String>("test", "key" + i, "value" + i);
       final Future<RecordMetadata> send = producer.send(record, new Callback() {
         public void onCompletion(RecordMetadata recordMetadata, Exception e) {
           if ( e != null ) {
             e.printStackTrace();
           } else {
             System.out.println("数据发送成功：" + record.key() + "," + record.value());
           }
         }
       });
     }

   } catch ( Exception e ) {

     e.printStackTrace();

   } finally {

     if ( producer != null ) {
       producer.close();
     }

   }
  }
}
```
### 2.4.5 消息可靠性

对于生产者发送的数据，我们有的时候是不关心数据是否已经发送成功的，我们只要发送就可以了。在这种场景中，消息可能会因为某些故障或问题导致丢失，我们将这种情况称之为消息不可靠。虽然消息数据可能会丢失，但是在某些需要高吞吐，低可靠的系统场景中，这种方式也是可以接受的，甚至是必须的。

但是在更多的场景中，我们是需要确定数据是否已经发送成功了且Kafka正确接收到数据的，也就是要保证数据不丢失，这就是所谓的消息可靠性保证。

而这个确定的过程一般是通过Kafka给我们返回的响应确认结果（Acknowledgement）来决定的，这里的响应确认结果我们也可以简称为ACK应答。根据场景，Kafka提供了3种应答处理，可以通过配置对象进行配置

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps72.jpg) 

#### 2.4.5.1ACK = 0（发送到网络就继续响应）

当生产数据时，生产者对象将数据通过网络客户端将数据发送到网络数据流中的时候，Kafka就对当前的数据请求进行了响应（确认应答），如果是同步发送数据，此时就可以发送下一条数据了。如果是异步发送数据，回调方法就会被触发。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps73.jpg) 

通过图形，明显可以看出，这种应答方式，数据已经走网络给Kafka发送了，但这其实并不能保证Kafka能正确地接收到数据，在传输过程中如果网络出现了问题，那么数据就丢失了。也就是说这种应答确认的方式，数据的可靠性是无法保证的。不过相反，因为无需等待Kafka服务节点的确认，通信效率倒是比较高的，也就是系统吞吐量会非常高。

#### 2.4.5.2 ACK = 1（写入leader就响应）

当生产数据时，Kafka Leader副本将数据接收到并写入到了日志文件后，就会对当前的数据请求进行响应（确认应答），如果是同步发送数据，此时就可以发送下一条数据了。如果是异步发送数据，回调方法就会被触发。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps74.jpg) 

通过图形，可以看出，这种应答方式，数据已经存储到了分区Leader副本中，那么数据相对来讲就比较安全了，也就是可靠性比较高。之所以说相对来讲比较安全，就是因为现在只有一个节点存储了数据，**而数据并没有来得及进行备份到follower副本，那么一旦当前存储数据的broker节点出现了故障，数据也依然会丢失。**

#### 2.4.5.3ACK = -1(默认，保存到副本就响应)

当生产数据时，Kafka Leader副本和Follower副本都已经将数据接收到并写入到了日志文件后，再对当前的数据请求进行响应（确认应答），如果是同步发送数据，此时就可以发送下一条数据了。如果是异步发送数据，回调方法就会被触发。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps75.jpg) 

通过图形，可以看出，这种应答方式，数据已经同时存储到了分区Leader副本和follower副本中，那么数据已经非常安全了，可靠性也是最高的。此时，如果Leader副本出现了故障，那么follower副本能够开始起作用，因为数据已经存储了，所以数据不会丢失。

不过这里需要注意，如果假设我们的分区有5个follower副本，编号为1，2，3，4，5

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps76.jpg) 

但是此时只有3个副本处于和Leader副本之间处于数据同步状态，那么此时分区就存在一个同步副本列表，我们称之为In Syn Replica，简称为ISR。此时，Kafka只要保证ISR中所有的4个副本接收到了数据，就可以对数据请求进行响应了。无需5个副本全部收到数据。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps77.jpg) 

### 2.4.6 消息去重 & 有序

#### 2.4.6.1数据重试

由于网络或服务节点的故障，Kafka在传输数据时，可能会导致数据丢失，所以我们才会设置ACK应答机制，尽可能提高数据的可靠性。但其实在某些场景中，数据的丢失并不是真正地丢失，而是“虚假丢失”，比如咱们将ACK应答设置为1，也就是说一旦Leader副本将数据写入文件后，Kafka就可以对请求进行响应了。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps78.jpg) 

此时，如果假设由于网络故障的原因，Kafka并没有成功将ACK应答信息发送给Producer，那么此时对于Producer来讲，以为kafka没有收到数据，所以就会一直等待响应，一旦超过某个时间阈值，就会发生超时错误，也就是说在Kafka Producer眼里，数据已经丢了

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps79.jpg) 

所以在这种情况下，kafka Producer会尝试对超时的请求数据进行重试(retry)操作。通过重试操作尝试将数据再次发送给Kafka。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps80.jpg) 

如果此时发送成功，那么Kafka就又收到了数据，而这两条数据是一样的，也就是说，导致了数据的重复。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps81.jpg) 

#### 2.4.6.2数据乱序

数据重试(**retry**)功能除了可能会导致数据重复以外，还可能会导致数据乱序。假设我们需要将编号为1，2，3的三条连续数据发送给Kafka。每条数据会对应于一个连接请求

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps82.jpg) 

此时，如果第一个数据的请求出现了故障，而第二个数据和第三个数据的请求正常，那么Broker就收到了第二个数据和第三个数据，并进行了应答。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps83.jpg) 

为了保证数据的可靠性，此时，Kafka Producer会将第一条数据重新放回到缓冲区的第一个。进行重试操作

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps84.jpg) 

如果重试成功，Broker收到第一条数据，你会发现。数据的顺序已经被打乱了。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps85.jpg) 

#### 2.4.6.3数据幂等性

为了解决Kafka传输数据时，所产生的数据重复和乱序问题，Kafka引入了幂等性操作，所谓的幂等性，就是Producer同样的一条数据，无论向Kafka发送多少次，kafka都只会存储一条。注意，这里的同样的一条数据，指的不是内容一致的数据，而是指的不断重试的数据。

当然kafka的幂等性智能保证一个分区的数据不重复

默认幂等性是不起作用的，所以如果想要使用幂等性操作，只需要在生产者对象的配置中开启幂等性配置即可

| **配置项**                                | **配置值** | **说明**                                   |
| ----------------------------------------------- | ---------------- | ------------------------------------------------ |
| enable.idempotence                    | true             | 开启幂等性                                       |
| max.in.flight.requests.per.connection\| 小于等于5        | 每个连接的在途请求数，不能大于5，取值范围为[1,5] |
| acks                               | all(-1)          | 确认应答，固定值，不能修改                       |
| retries                            | >0               | 重试次数，推荐使用Int最大值                      |

kafka是如何实现数据的幂等性操作呢，我们这里简单说一下流程：

(1) 开启幂等性后，为了保证数据不会重复，那么就需要给每一个请求批次的数据增加唯一性标识，kafka中，这个标识采用的是连续的序列号数字sequencenum，但是不同的生产者Producer可能序列号是一样的，所以仅仅靠seqnum还无法唯一标记数据，所以还需要同时对生产者进行区分，所以Kafka采用申请生产者ID（producerid）的方式对生产者进行区分。这样，在发送数据前，我们就需要提前申请producerid以及序列号sequencenum

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps86.jpg) 

(2) Broker中会给每一个分区记录生产者的生产状态：采用队列的方式缓存最近的5个批次数据。队列中的数据按照seqnum进行升序排列。这里的数字5是经过压力测试，均衡空间效率和时间效率所得到的值，所以为固定值，无法配置且不能修改。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps87.jpg) 

(3) 如果Borker当前新的请求批次数据在缓存的5个旧的批次中存在相同的，如果有相同的，那么说明有重复，当前批次数据不做任何处理。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps88.jpg) 

(4) 如果Broker当前的请求批次数据在缓存中没有相同的，那么判断当前新的请求批次的序列号是否为缓存的最后一个批次的序列号加1，如果是，说明是连续的，顺序没乱。那么继续，如果不是，那么说明数据已经乱了，发生异常。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps89.jpg) 

(5) Broker根据异常返回响应，通知Producer进行重试。Producer重试前，需要在缓冲区中将数据重新排序，保证正确的顺序后。再进行重试即可。

(6) 如果请求批次不重复，且有序，那么更新缓冲区中的批次数据。将当前的批次放置再队列的结尾，将队列的第一个移除，保证队列中缓冲的数据最多5个。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps90.jpg) 

从上面的流程可以看出，Kafka的幂等性是通过消耗时间和性能的方式提升了数据传输的有序和去重，在一些对数据敏感的业务中是十分重要的。但是通过原理，咱们也能明白，这种幂等性还是有缺陷的：

幂等性的producer仅做到单分区上的幂等性，即单分区消息有序不重复，多分区无法保证幂等性。

只能保持生产者单个会话的幂等性，无法实现跨会话的幂等性，也就是说如果一个producer挂掉再重启，那么重启前和重启后的producer对象会被当成两个独立的生产者，从而获取两个不同的独立的生产者ID，导致broker端无法获取之前的状态信息，所以无法实现跨会话的幂等。要想解决这个问题，可以采用后续的事务功能。

#### 2.4.6.4数据事务

对于幂等性的缺陷，kafka可以采用事务的方式解决跨会话的幂等性。基本的原理就是通过事务功能管理生产者ID，保证事务开启后，生产者对象总能获取一致的生产者ID。

为了实现事务，Kafka引入了事务协调器（TransactionCoodinator）负责事务的处理，所有的事务逻辑包括分派PID等都是由TransactionCoodinator负责实施的。TransactionCoodinator 会将事务状态持久化到该主题中。

事务基本的实现思路就是通过配置的事务ID，将生产者ID进行绑定，然后存储在Kafka专门管理事务的内部主题 __transaction_state\中，而内部主题的操作是由事务协调器（TransactionCoodinator）对象完成的，这个协调器对象有点类似于咱们数据发送时的那个副本Leader。其实这种设计是很巧妙的，因为kafka将事务ID和生产者ID看成了消息数据，然后将数据发送到一个内部主题中。这样，使用事务处理的流程和咱们自己发送数据的流程是很像的。接下来，我们就把这两个流程简单做一个对比。

##### 2.4.6.4.1 普通数据发送流程

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps91.jpg) 

##### 2.4.6.4.2 事务数据发送流程

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps92.jpg) 

通过两张图大家可以看到，基本的事务操作和数据操作是很像的，不过要注意，我们这里只是简单对比了数据发送的过程，其实它们的区别还在于数据发送后的提交过程。普通的数据操作，只要数据写入了日志，那么对于消费者来讲。数据就可以读取到了，但是事务操作中，如果数据写入了日志，但是没有提交的话，其实数据默认情况下也是不能被消费者看到的。只有提交后才能看见数据。

##### 2.4.6.4.3 事务提交流程

Kafka中的事务是分布式事务，所以采用的也是二阶段提交

 第一个阶段提交事务协调器会告诉生产者事务已经提交了，所以也称之预提交操作，事务协调器会修改事务为预提交状态

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps93.jpg) 

 第二个阶段提交事务协调器会向分区Leader节点中发送数据标记，通知Broker事务已经提交，然后事务协调器会修改事务为完成提交状态
![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps94.jpg) 

特殊情况下，事务已经提交成功，但还是读取不到数据，那是因为当前提交成功只是一阶段提交成功，事务协调器会继续向各个Partition发送marker信息，此操作会无限重试，直至成功。

但是不同的Broker可能无法全部同时接收到marker信息，此时有的Broker上的数据还是无法访问，这也是正常的，因为kafka的事务不能保证强一致性，只能保证最终数据的一致性，无法保证中间的数据是一致的。不过对于常规的场景这里已经够用了，事务协调器会不遗余力的重试，直至成功。

总体

![image-20240512211107622](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/image-20240512211107622.png)

##### 2.4.6.4.4 事务操作代码 

```


import org.apache.kafka.clients.producer.*;

import org.apache.kafka.common.serialization.StringSerializer;

 

import java.util.HashMap;

import java.util.Map;

import java.util.concurrent.Future;

 

public class ProducerTransactionTest {

  public static void main(String[] args) {

   Map<String, Object> configMap = new HashMap<>();

  configMap.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   configMap.put( ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

  configMap.put( ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

  // TODO 配置幂等性

  configMap.put( ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);

  // TODO 配置事务ID

  configMap.put( ProducerConfig.TRANSACTIONAL_ID_CONFIG, "my-tx-id");

  // TODO 配置事务超时时间

   configMap.put( ProducerConfig.TRANSACTION_TIMEOUT_CONFIG, 5);

   // TODO 创建生产者对象

   KafkaProducer<String, String> producer = new KafkaProducer<>(configMap);

   // TODO 初始化事务

   producer.initTransactions();

   try {

     // TODO 启动事务

     producer.beginTransaction();

     // TODO 生产数据

     for ( int i = 0; i < 10; i++ ) {

       ProducerRecord<String, String> record = new ProducerRecord<String, String>("test", "key" + i, "value" + i);

       final Future<RecordMetadata> send = producer.send(record);

    }

    // TODO 提交事务

     producer.commitTransaction();

  } catch ( Exception e ) {

     e.printStackTrace();

    // TODO 终止事务

    producer.abortTransaction();

  }

  // TODO 关闭生产者对象

  producer.close();

 

  }

}
```
#### 2.4.6.5数据传输语义

| 传输语义      | 说明                                                         | 例子             |
| ------------- | ------------------------------------------------------------ | ---------------- |
| at most once  | 最多一次：不管是否能接收到，数据最多只传一次。这样数据可能会丢失， | Socket， ACK=0   |
| at least once | 最少一次：消息不会丢失，如果接收不到，那么就继续发，所以会发送多次，直到收到为止，有可能出现数据重复 | ACK=1            |
| Exactly once  | 精准一次：消息只会一次，不会丢，也不会重复。       | 幂等+事务+ACK=-1 |

#### 2.4.7 数据收集与发送线程

![image-20240508222443178](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/image-20240508222443178.png)

数据发送到一个partion之后，不会立即发送，会累计数据到16K就会通过发送线程发送，如果数据超过了16K，就会锁定。

当然，如果超过一个时间限制，也会直接发送，无需等待积累到16K。

发送进程调用NIO网络模型发送网络消息。

Sender线程是一个独立的线程，这个线程发送数据并且得到应答

## 2.5 存储消息

数据已经由生产者Producer发送给Kafka集群，当Kafka接收到数据后，会将数据写入本地文件中。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps96.jpg) 

### 2.5.1 存储组件

KafkaApis : Kafka应用接口组件，当Kafka Producer向Kafka Broker发送数据请求后，Kafka Broker接收请求，会使用Apis组件进行请求类型的判断，然后选择相应的方法进行处理。

ReplicaManager: 副本管理器组件，用于提供主题副本的相关功能，在数据的存储前进行ACK校验和事务检查，并提供数据请求的响应处理

Partition : 分区对象，主要包含分区状态变换的监控，分区上下线的处理等功能，在数据存储是主要用于对分区副本数量的相关校验，并提供追加数据的功能

UnifiedLog : 同一日志管理组件，用于管理数据日志文件的新增，删除等功能，并提供数据日志文件偏移量的相关处理。

LocalLog : 本地日志组件，管理整个分区副本的数据日志文件。假设当前主题分区中有3个日志文件，那么3个文件都会在组件中进行管理和操作。

LogSegment: 文件段组件，对应具体的某一个数据日志文件，假设当前主题分区中有3个日志文件，那么3个文件每一个都会对应一个LogSegment组件，并打开文件的数据管道FileChannel。数据存储时，就是采用组件中的FileChannel实现日志数据的追加

LogConfig: 日志配置对象，常用的数据存储配置

| 参数名           | 参数作用      | 类型 | 默认值                 | 推荐值 |
| --------------------------- | -------------------------- | -------------- | -------------------------------- | ---------------- |
| min.insync.replicas         | 最小同步副本数量           | 推荐           | 1                                | 2                |
| log.segment.bytes           | 文件段字节数据大小限制     | 可选           | 1G = 1024*1024*1024 byte         |                  |
| log.roll.hours              | 文件段强制滚动时间阈值     | 可选           | 7天 =24 * 7 * 60 * 60 * 1000L ms |                  |
| log.flush.interval.messages | 满足刷写日志文件的数据条数 | 可选           | Long.MaxValue                    | 不推荐           |
| log.flush.interval.ms       | 满足刷写日志文件的时间周期 | 可选           | Long.MaxValue                    | 不推荐           |
| log.index.interval.bytes    | 刷写索引文件的字节数       | 可选           | 4 * 1024                         |                  |
| replica.lag.time.max.ms     | 副本延迟同步时间           | 可选           | 30s                              |                  |

### 2.5.2 数据存储

Kafka Broker节点从获取到生产者的数据请求到数据存储到文件的过程相对比较简单，只是中间会进行一些基本的数据检查和校验。所以接下来我们就将数据存储的基本流程介绍一下：

#### 2.5.2.1 ACKS校验

Producer将数据发送给Kafka Broker时，会告知Broker当前生产者的数据生产场景，从而要求Kafka对数据请求进行应答响应确认数据的接收情况，Producer获取应答后可以进行后续的处理。这个数据生产场景主要考虑的就是数据的可靠性和数据发送的吞吐量。由此，Kafka将生产场景划分为3种不同的场景：

ACKS = 0: Producer端将数据发送到网络输出流中，此时Kafka就会进行响应。在这个场景中，数据的应答是非常快的，但是因为仅仅将数据发送到网络输出流中，所以是无法保证kafka broker节点能够接收到消息，假设此时网络出现抖动不稳定导致数据丢失，而由于Kafka已经做出了确认收到的应答，所以此时Producer端就不会再次发送数据，而导致数据真正地丢失了。所以此种场景，数据的发送是不可靠的。

ACKS = 1: Producer端将数据发送到Broker中，并保存到当前节点的数据日志文件中，Kafka就会进行确认收到数据的响应。因为数据已经保存到了文件中，也就是进行了持久化，那么相对于ACKS=0，数据就更加可靠。但是也要注意，因为Kafka是分布式的，所以集群的运行和管理是非常复杂的，难免当前Broker节点出现问题而宕掉，那么此时，消费者就消费不到我们存储的数据了，此时，数据我们还是会认为丢失了。

ACKS = -1（all）: Kafka在管理分区时，会了数据的可靠性和更高的吞吐量，提供了多个副本，而多个副本之间，会选出一个副本作为数据的读写副本，称之为Leader领导副本，而其他副本称之Follower追随副本。普通场景中，所有的这些节点都是需要保存数据的。而Kafka会优先将Leader副本的数据进行保存，保存成功后，再由Follower副本向Leader副本拉取数据，进行数据同步。一旦所有的这些副本数据同步完毕后，Kafka再对Producer进行收到数据的确认。此时ACKS应答就是-1（all）。明显此种场景，多个副本本地文件都保存了数据，那么数据就更加可靠，但是相对，应答时间更长，导致Kafka吞吐量降低。

基于上面的三种生产数据的场景，在存储数据前，需要校验生产者需要的应答场景是否合法有效。

#### 2.5.2.2内部主题校验

Producer向Kafka Broker发送数据时，是必须指定主题Topic的，但是这个主题的名称不能是kafka的内部主题名称。Kafka为了管理的需要，创建了2个内部主题，一个是用于事务处理的__transaction_state内部主题，还有一个是用于处理消费者偏移量的__consumer_offsets内部主题。生产者是无法对这两个主题生产数据的，所以在存储数据之前，需要对主题名称进行校验有效性校验。

#### 2.5.2.3 ACKS应答及副本数量关系校验

Kafka为了数据可靠性更高一些，需要分区的所有副本都能够存储数据，但是分布式环境中难免会出现某个副本节点出现故障，暂时不能同步数据。在Kafka中，能够进行数据同步的所有副本，我们称之为In Sync Replicas，简称ISR列表。

当生产者Producer要求的数据ACKS应答为-1的时候，那么就必须保证能够同步数据的所有副本能够将数据保存成功后，再进行数据的确认应答。但是一种特殊情况就是，如果当前ISR列表中只有一个Broker存在，那么此时只要这一个Broker数据保存成功了，那么就产生确认应答了，数据依然是不可靠的，那么就失去了设置ACK=all的意义了，所以此时还需要对ISR列表中的副本数量进行约束，至少不能少于2个。这个数量是可以通过配置文件配置的。参数名为：min.insync.replicas。默认值为1（不推荐）

所以存储数据前，也需要对ACK应答和最小分区副本数量的关系进行校验。

#### 2.5.2.4 日志文件滚动判断

数据存储到文件中，如果数据文件太大，对于查询性能是会有很大影响的，所以副本数据文件并不是一个完整的大的数据文件，而是根据某些条件分成很多的小文件，每个小文件我们称之为文件段。其中的一个条件就是文件大小，参数名为：log.segment.bytes。默认值为1G。如果当前日志段剩余容量可能无法容纳新消息集合，因此有必要创建一个新的日志段来保存待写入的所有消息。此时日志文件就需要滚动生产新的。

除了文件大小外，还有时间间隔，如果文件段第一批数据有时间戳，那么当前批次数据的时间戳和第一批数据的时间戳间隔大于滚动阈值，那么日志文件也会滚动生产新的。如果文件段第一批数据没有时间戳，那么就用当前时间戳和文件创建时间戳进行比对，如果大于滚动阈值，那么日志文件也会滚动生产新的。这个阈值参数名为：log.roll.hours，默认为7天。如果时间到达，但是文件不满1G，依然会滚动生产新的数据文件。

如果索引文件或时间索引文件满了，或者索引文件无法存放当前索引数据了，那么日志文件也会滚动生产新的。

基于以上的原则，需要在保存数据前进行判断。

#### 2.5.2.5 请求数据重复性校验

因为Kafka允许生产者进行数据重试操作，所以因为一些特殊的情况，就会导致数据请求被Kafka重复获取导致数据重复，所以为了数据的幂等性操作，需要在Broker端对数据进行重复性校验。这里的重复性校验只能对同一个主题分区的5个在途请求中数据进行校验，所以需要在生产者端进行相关配置。

#### 2.5.2.6 请求数据序列号校验

因为Kafka允许生产者进行数据重试操作，所以因为一些特殊的情况，就会导致数据请求被Kafka重复获取导致数据顺序发生改变从而引起数据乱序。为了防止数据乱序，需要在Broker端对数据的序列号进行连续性（插入数据序列号和Broker缓冲的最后一个数据的序列号差值为1）校验。

#### 2.5.2.7 数据存储

将数据通过LogSegment中FileChannel对象。将数据写入日志文件，写入完成后，更新当前日志文件的数据偏移量。

### 2.5. 存储文件格式

我们已经将数据存储到了日志文件中，当然除了日志文件还有其他的一些文件，所以接下来我们就了解一下这些文件：

#### 2.5.3.1 数据日志文件（log文件）

Kafka系统早期设计的目的就是日志数据的采集和传输，所以数据是使用log文件进行保存的。我们所说的数据文件就是以.log作为扩展名的日志文件。文件名长度为20位长度的数字字符串，数字含义为当前日志文件的第一批数据的基础偏移量，也就是文件中保存的第一条数据偏移量。字符串数字位数不够的，前面补0。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps97.jpg) 

我们的常规数据主要分为两部分：批次头 + 数据体

##### 2.5.3.1.1 批次头

| 数据项                      | 含义       | 长度 |
| --------------------------------------- | --------------------- | -------------- |
| BASE_OFFSET_OFFSET            | 基础偏移量偏移量      | 8              |
| LENGTH_OFFSET                 | 长度偏移量            | 4              |
| PARTITION_LEADER_EPOCH_OFFSET | Leaader分区纪元偏移量 | 4              |
| MAGIC_OFFSET                  | 魔数偏移量            | 1              |
| ATTRIBUTES_OFFSET             | 属性偏移量            | 2              |
| BASE_TIMESTAMP_OFFSET         | 基础时间戳偏移量      | 8              |
| MAX_TIMESTAMP_OFFSET          | 最大时间戳偏移量      | 8              |
| LAST_OFFSET_DELTA_OFFSET      | 最后偏移量偏移量      | 4              |
| PRODUCER_ID_OFFSET            | 生产者ID偏移量        | 8              |
| PRODUCER_EPOCH_OFFSET         | 生产者纪元偏移量      | 2              |
| BASE_SEQUENCE_OFFSET          | 基础序列号偏移量      | 4              |
| RECORDS_COUNT_OFFSET          | 记录数量偏移量        | 4              |
| CRC_OFFSET                    | CRC校验偏移量         | 4              |

批次头总的字节数为：61 byte

##### 2.5.3.1.2 数据体

| 数据项            | 含义                     | 长度 |
| --------------------------- | ---------------------------------- | -------------- |
| size              | 固定值                             | 1              |
| offsetDelta       | 固定值                             | 1              |
| timestampDelta    | 时间戳                             | 1              |
| keySize           | Key字节长度                        | 1（动态）      |
| keySize(Varint)   | Key变量压缩长度算法需要大小        | 1（动态）      |
| valueSize         | value字节长度                      | 1（动态）      |
| valueSize(Varint) | Value变量压缩长度算法需要大小      | 1（动态）      |
| Headers           | 数组固定长度                       | 1（动态）      |
| sizeInBytes       | 上面长度之和的压缩长度算法需要大小 | 1              |

表中的后5个值为动态值，需要根据数据的中key，value变化计算得到。此处以数据key=key1，value=value1为例。

压缩长度算法：

```
中间值1 = (算法参数 << 1) ^ (算法参数 >> 31));
中间值2 = Integer.numberOfLeadingZeros(中间值1);
结果   = (38 - 中间值2) / 7 + 中间值2 / 32;
```

例如：

```
假设当前key为：key1，调用算法时，参数为key.length = 4
中间值1 = (4<<1) ^ (4>>31) = 8
中间值2 = Integer.numberOfLeadingZeros(8) = 28
结果   = (38-28)/7 + 28/32 = 1 + 0 = 1
所以如果key取值为key1,那么key的变长长度就是1
按照上面的计算公式可以计算出，如果我们发送的数据是一条为（key1，value1）的数据， 那么Kafka当前会向日志文件增加的数据大小为：
```

**追加数据字节计算**

```
批次头 = 61
数据体 = 1 + 1 + 1 + 4 + 1 + 6 + 1 + 1 + 1 = 17
总的字节大小为61 + 17 = 78
```

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps98.jpg) 

**如果我们发送的数据是两条为（key1，value1），（key2，value2）的数据**， 那么Kafka当前会向日志文件增加的数据大小为：

**追加数据字节计算**

第一条数据：

批次头 = 61

数据体 = 1 + 1 + 1 + 4 + 1 + 6 + 1 + 1 + 1 = 17

第二条数据：

**因为字节少，没有满足批次要求，所以两条数据是在一批中的，那么批次头不会重新计算，直接增加数据体即可**

数据体 = 1 + 1 + 1 + 4 + 1 + 6 + 1 + 1 + 1 = 17

总的字节大小为61 + 17 + 17 = 95

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps99.jpg) 

##### 2.5.3.1.3 数据含义

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps100.jpg) 

| 数据项     | 含义                                               |
| -------------------- | ------------------------------------------------------------ |
| baseOffset           | 当前batch中第一条消息的位移                                  |
| lastOffset           | 最新消息的位移相对于第一条消息的唯一增量                     |
| count                | 当前batch有的数据数量，kafka在进行消息遍历的时候，可以通过该字段快速的跳跃到下一个batch进行数据读取 |
| partitionLeaderEpoch | 记录了当前消息所在分区的 leader 的服务器版本（纪元），主要用于进行一些数据版本的校验和转换工作 |
| crc                  | 当前整个batch的数据crc校验码，主要用于对数据进行差错校验的   |
| compresscode         | 数据压缩格式，主要有GZIP、LZ4、Snappy、zstd四种              |
| baseSequence         | 当前批次中的基础序列号                                       |
| lastSequence         | 当前批次中的最后一个序列号                                   |
| producerId           | 生产者ID                                                     |
| producerEpoch        | 记录了当前消息所在分区的Producer的服务器版本（纪元）         |
| isTransactional      | 是否开启事务                                                 |
| magic                | 魔数（Kafka服务程序协议版本号）                              |
| CreateTime（data）   | 数据创建的时间戳                                             |
| isControl            | 控制类数据（produce的数据为false，事务Marker为true）         |
| compresscodec        | 压缩格式，默认无                                             |
| isvalid              | 数据是否有效                                                 |
| offset               | 数据偏移量，从0开始                                          |
| key                  | 数据key                                                      |
| payload              | 数据value                                                    |
| sequence             | 当前批次中数据的序列号                                       |
| CreateTime（header） | 当前批次中最后一条数据的创建时间戳                           |

#### 2.5.3.2 数据索引文件（index结尾的文件）



Kafka的基础设置中，数据日志文件到达1G才会滚动生产新的文件。那么从1G文件中想要快速获取我们想要的数据，效率还是比较低的。通过前面的介绍，如果我们能知道数据在文件中的位置（position），那么定位数据就会快很多，问题在于我们如何才能在知道这个位置呢。

Kafka在存储数据时，都会保存数据的偏移量信息，而偏移量是从0开始计算的。简单理解就是数据的保存顺序。比如第一条保存的数据，那么偏移量就是0，第二条保存的数据偏移量就是1，但是这个偏移量只是告诉我们数据的保存顺序，却无法定位数据，不过需要注意的是，每条数据的大小是可以确定的（参考上一个小节的内容）。既然可以确定，那么数据存放在文件的位置起始也就是确定了，所以Kafka在保存数据时，其实是可以同时保存位置的，那么我们在访问数据时，只要通过偏移量其实就可以快速定位日志文件的数据了。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps101.jpg) 

不过这依然有问题，就是数据量太多了，对应的偏移量也太多了，并且主题分区的数据文件会有很多，那我们是如何知道数据在哪一个文件中呢？为了定位方便Kafka在提供日志文件保存数据的同时，还提供了用于数据定位的索引文件，索引文件中保存的就是逻辑偏移量和数据物理存储位置（偏移量）的对应关系。并且还记得吗?每个数据日志文件的名称就是当前文件中数据䣌起始偏移量，所以通过偏移量就可以快速选取文件以及定位数据的位置从而快速找到数据。这种感觉就有点像Java的HashMap通过Key可以快速找到Value的感觉一样，如果不知道Key，那么从HashMap中获取Value是不是就特别慢。道理是一样的。

Kafka的数据索引文件都保存了什么呢？咱们来看一下：

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps102.jpg) 

通过图片可以看到，索引文件中保存的就是逻辑偏移量和物理偏移量位置的关系。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps103.jpg) 

有了这个索引文件，那么我们根据数据的顺序获取数据就非常的方便和高效了。不过，相信大家也注意到了，那就是索引文件中的offset并不连续。那如果我想获取offset等于3的数据怎么办？其实也不难，因为offset等于3不就是offset等于2的一下条吗？那我使用offset等于2的数据的position + size不就定位了offset等于3的位置了吗，当然了我举得例子有点过于简单了，不过本质确实差的不多，kafka在查询定位时其实采用的就是二分查找法。

不过，为什么Kafka的索引文件是不连续的呢，那是因为如果每条数据如果都把偏移量的定位保存下来，数据量也不小，还有就是，如果索引数据丢了几条，其实并不会太影响查询效率，比如咱们之前举得offset等于3的定位过程。因为Kafka底层实现时，采用的是虚拟内存映射技术mmap，将内存和文件进行双向映射，操作内存数据就等同于操作文件，所以效率是非常高的，但是因为是基于内存的操作，所以并不稳定，容易丢数据，因此Kafka的索引文件中的索引信息是不连续的，而且为了效率，kafka默认情况下，4kb的日志数据才会记录一次索引，但是这个是可以进行配置修改的，参数为log.index.interval.bytes，默认值为4096。所以我们有的时候会将kafka的不连续索引数据称之为稀疏索引。

##### 简述索引文件的查找过程

首先会通过一个跳跃map来寻找对应的索引段，即找到对应的index文件，文件有跳跃的时候会利用floorEntry方法找到左边的段。

![image-20240512162121058](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/image-20240512162121058.png)

通过索引文件的内容找到具体的内容，使用的是二分查找法，具体看上面，同时，由于每个数据的批次与数据都是固定的，可以通过字节计算的方式定位到具体的数据。

![image-20240512162434948](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/image-20240512162434948.png)

索引文件的段是一种稀疏索引，由于kafka一般不会去查找一条数据，通常是批量查找，因此kafka通过稀疏索引查找数据方式是合理的。





#### 2.5.3.3 数据时间索引文件（timeindex文件）

某些场景中，我们不想根据顺序（偏移量）获取Kafka的数据，而是想根据时间来获取的数据。这个时候，可没有对应的偏移量来定位数据，那么查找的效率就非常低了，因为kafka还提供了时间索引文件，咱们来看看它的内容是什么

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps104.jpg) 

通过图片，大家可以看到，这个时间索引文件起始就是将时间戳和偏移量对应起来了，那么此时通过时间戳就可以找到偏移量，再通过偏移量找到定位信息，再通过定位信息找到数据不就非常方便了吗。

#### 2.5.3.4 查看文件内容

如果我们想要查看文件的内容，直接看是看不了，需要采用特殊的之类

\# 进入bin/windows目录

cd bin/windows

\# 执行查看文件的指令

kafka-run-class.bat kafka.tools.DumpLogSegments --files ../../data/kafka/test-0/00000000000000000000.log --print-data-log

### 2.5.4 数据刷写

在Linux系统中，当我们把数据写入文件系统之后，其实数据在操作系统的PageCache（页缓冲）里面，并没有刷到磁盘上。如果操作系统挂了，数据就丢失了。一方面，应用程序可以调用fsync这个系统调用来强制刷盘，另一方面，操作系统有后台线程，定时刷盘。频繁调用fsync会影响性能，需要在性能和可靠性之间进行权衡。实际上，Kafka提供了参数进行数据的刷写

 log.flush.interval.messages ：达到消息数量时，会将数据flush到日志文件中。

 log.flush.interval.ms ：间隔多少时间(ms)，执行一次强制的flush操作。

flush.scheduler.interval.ms：所有日志刷新到磁盘的频率

log.flush.interval.messages和log.flush.interval.ms无论哪个达到，都会flush。官方不建议通过上述的三个参数来强制写盘，数据的可靠性应该通过replica来保证，而强制flush数据到磁盘会对整体性能产生影响。

### 2.5.5 副本同步

Kafka中，分区的某个副本会被指定为 Leader，负责响应客户端的读写请求。分区中的其他副本自动成为 Follower，主动拉取（同步）Leader 副本中的数据，写入自己本地日志，确保所有副本上的数据是一致的。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps105.jpg) 

#### 2.5.6.1 启动数据同步线程

Kafka创建主题时，会根据副本分配策略向指定的Broker节点发出请求，将不同的副本节点设定为Leader或Follower。一旦某一个Broker节点设定为Follower节点，那么Follower节点会启动数据同步线程ReplicaFetcherThread，从Leader副本节点同步数据。

线程运行后，会不断重复两个操作：截断（truncate）和抓取（fetch）。

 截断：为了保证分区副本的数据一致性，当分区存在Leader Epoch值时，会将副本的本地日志截断到Leader Epoch对应的最新位移处.如果分区不存在对应的 Leader Epoch 记录，那么依然使用原来的高水位机制，将日志调整到高水位值处。



抓取：向Leader同步最新的数据。





#### 2.5.6.2 生成数据同步请求

启动线程后，需要周期地向Leader节点发送FETCH请求，用于从Leader获取数据。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps106.jpg) 

等待Leader节点的响应的过程中，会阻塞当前同步数据线程。

#### 2.5.6.3 处理数据响应

当Leader副本返回响应数据时，其中会包含多个分区数据，当前副本会遍历每一个分区，将分区数据写入数据文件中。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps107.jpg) 

#### 2.5.6.4 更新数据偏移量

当Leader副本返回响应数据时，除了包含多个分区数据外，还包含了和偏移量相关的数据HW和LSO，副本需要根据场景对Leader返回的不同偏移量进行更新。

##### 2.5.6.4.1 Offset

Kafka的每个分区的数据都是有序的，所谓的数据偏移量，指的就是Kafka在保存数据时，用于快速定位数据的标识，类似于Java中数组的索引，从0开始。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps108.jpg) 

Kafka的数据文件以及数据访问中包含了大量和偏移量的相关的操作。

##### 2.5.6.4.2 LSO

起始偏移量（Log Start Offset），每个分区副本都有起始偏移量，用于表示副本数据的起始偏移位置，初始值为0。

LSO一般情况下是无需更新的，但是如果数据过期，或用户手动删除数据时，Leader的Log Start Offset可能发生变化，Follower副本的日志需要和Leader保持严格的一致，因此，如果Leader的该值发生变化，Follower自然也要发生变化保持一致。

##### 2.5.6.4.3 LEO

日志末端位移（Log End Offset），**表示下一条待写入消息的offset**，每个分区副本都会记录自己的LEO。对于Follower副本而言，它能读取到Leader副本 LEO 值以下的所有消息。

对比所有副本的LEO来确定水位线

##### 2.5.6.4.1 HW

高水位值（High Watermark），定义了消息可见性，标识了一个特定的消息偏移量（offset），消费者只能拉取到这个水位offset之前的消息，同时这个偏移量还可以帮助Kafka完成副本数据同步操作。

需要注意水位线的含义：

![image-20240512170758479](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/image-20240512170758479.png)

水位线以下的数据是已经在所有副本同步的数据，消费者能看到的就是低于水位线的数据。

水位线随着flower的不断同步而不断上涨。

### 2.5.6 数据一致性

#### 2.5.6.1数据一致性

Kafka的设计目标是：高吞吐、高并发、高性能。为了做到以上三点，它必须设计成分布式的，多台机器可以同时提供读写，并且需要为数据的存储做冗余备份。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps109.jpg) 

图中的主题有3个分区，每个分区有3个副本，这样数据可以冗余存储，提高了数据的可用性。并且3个副本有两种角色，Leader和Follower，Follower副本会同步Leader副本的数据。

一旦Leader副本挂了，Follower副本可以选举成为新的Leader副本， 这样就提升了分区可用性，但是相对的，在提升了分区可用性的同时，也就牺牲了数据的一致性。

我们来看这样的一个场景：一个分区有3个副本，一个Leader和两个Follower。Leader副本作为数据的读写副本，所以生产者的数据都会发送给leader副本，而两个follower副本会周期性地同步leader副本的数据，但是因为网络，资源等因素的制约，同步数据的过程是有一定延迟的，所以3个副本之间的数据可能是不同的。具体如下图所示：

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps110.jpg) 

此时，假设leader副本因为意外原因宕掉了，那么Kafka为了提高分区可用性，此时会选择2个follower副本中的一个作为Leader对外提供数据服务。此时我们就会发现，对于消费者而言，之前leader副本能访问的数据是D，但是重新选择leader副本后，能访问的数据就变成了C，这样消费者就会认为数据丢失了，也就是所谓的数据不一致了。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps111.jpg) 

为了提升数据的一致性，Kafka引入了高水位（HW ：High Watermark）机制，Kafka在不同的副本之间维护了一个水位线的机制（其实也是一个偏移量的概念），消费者只能读取到水位线以下的的数据。这就是所谓的木桶理论：木桶中容纳水的高度，只能是水桶中最短的那块木板的高度。这里将整个分区看成一个木桶，其中的数据看成水，而每一个副本就是木桶上的一块木板，那么这个分区（木桶）可以被消费者消费的数据（容纳的水）其实就是数据最少的那个副本的最后数据位置（木板高度）。

也就是说，消费者一开始在消费Leader的时候，虽然Leader副本中已经有a、b、c、d 4条数据，但是由于高水位线的限制，所以也只能消费到a、b这两条数据。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps112.jpg) 

这样即使leader挂掉了，但是对于消费者来讲，消费到的数据其实还是一样的，因为它能看到的数据是一样的，也就是说，消费者不会认为数据不一致。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps113.jpg) 

不过也要注意，因为follower要求和leader的日志数据严格保持一致，所以就需要根据现在Leader的数据偏移量值对其他的副本进行数据截断（truncate）操作。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps114.jpg) 

#### 2.5.6.2 HW在副本之间的传递

HW高水位线会随着follower的数据同步操作，而不断上涨，也就是说，follower同步的数据越多，那么水位线也就越高，那么消费者能访问的数据也就越多。接下来，我们就看一看，follower在同步数据时HW的变化。

首先，初始状态下，Leader和Follower都没有数据，所以和偏移量相关的值都是初始值0，而由于Leader需要管理follower，所以也包含着follower的相关偏移量（LEO）数据。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps115.jpg) 

生产者向Leader发送两条数据，Leader收到数据后，会更新自身的偏移量信息。

Leader副本偏移量更新：

LEO=LEO+2=2

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps116.jpg) 

接下来，Follower开始同步Leader的数据，同步数据时，会将自身的LEO值作为参数传递给Leader。此时，Leader会将数据传递给Follower，且同时Leader会根据所有副本的LEO值更新HW。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps117.jpg) 

Leader副本偏移量更新：

HW = Math.max[HW, min(LeaderLEO，F1-LEO，F2-LEO)]=0

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps118.jpg) 

由于两个Follower的数据拉取速率不一致，所以Follower-1抓取了2条数据，而Follower-2抓取了1条数据。Follower再收到数据后，会将数据写入文件，并更新自身的偏移量信息。

Follower-1副本偏移量更新：

LEO=LEO+2=2

HW = Math.min[LeaderHW, LEO]=0

Follower-2副本偏移量更新：

LEO=LEO+1=1

HW = Math.min[LeaderHW, LEO]=0

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps119.jpg) 

接下来Leader收到了生产者的数据C，那么此时会根据相同的方式更新自身的偏移量信息

Leader副本偏移量更新：

LEO=LEO+1=3

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps120.jpg) 

follower接着向Leader发送Fetch请求，同样会将最新的LEO作为参数传递给Leader。Leader收到请求后，会更新自身的偏移量信息。

Leader副本偏移量更新：

HW = Math.max[HW, min(LeaderLEO，F1-LEO，F2-LEO)]=0

 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps121.jpg) 

此时，Leader会将数据发送给Follower，同时也会将HW一起发送。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps122.jpg) 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps123.jpg) 

Follower收到数据后，会将数据写入文件，并更新自身偏移量信息

Follower-1副本偏移量更新：

LEO=LEO+1=3

HW = Math.min[LeaderHW, LEO]=1

Follower-2副本偏移量更新：

LEO=LEO+1=2

HW = Math.min[LeaderHW, LEO]=1

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps124.jpg) 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps125.jpg) 

因为Follower会不断重复Fetch数据的过程，所以前面的操作会不断地重复。最终，follower副本和Leader副本的数据和偏移量是保持一致的。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps126.jpg) 

上面演示了副本列表ISR中Follower副本和Leader副本之间HW偏移量的变化过程，但特殊情况是例外的。比如当前副本列表ISR中，只剩下了Leader一个副本的场合下，是不需要等待其他副本的，直接推高HW即可。

#### 2.5.6.3 ISR（In-Sync Replicas）伸缩

**ISR存储着集群kafka的brokid列表，第一个代表着leader**

在Kafka中，一个Topic（主题）包含多个Partition（分区），Topic是逻辑概念，而Partition是物理分组。一个Partition包含多个Replica（副本），副本有两种类型Leader Replica/Follower Replica，Replica之间是一个Leader副本对应多个Follower副本。注意：分区数可以大于节点数，但副本数不能大于节点数。因为副本需要分布在不同的节点上，才能达到备份的目的。

Kafka的分区副本中只有Leader副本具有数据写入的功能，而Follower副本需要不断向Leader发出申请，进行数据的同步。这里所有同步的副本会形成一个列表，我们称之为同步副本列表（**In-Sync Replicas**），也可以简称ISR，除了ISR以外，还有已分配的副本列表（Assigned Replicas），简称AR。这里的AR其实不仅仅包含ISR，还包含了没有同步的副本列表（Out-of-Sync Replicas），简称OSR

生产者Producer生产数据时，ACKS应答机制如果设置为all（-1），那此时就需要保证同步副本列表ISR中的所有副本全部接收完毕后，Kafka才会进行确认应答。

数据存储时，只有ISR中的所有副本LEO数据都更新了，才有可能推高HW偏移量的值。这就可以看出，ISR在Kafka集群的管理中是非常重要的。

在Broker节点中，有一个副本管理器组件（ReplicaManager），除了读写副本、管理分区和副本的功能之外，还有一个重要的功能，那就是管理ISR。这里的管理主要体现在两个方面：

周期性地查看 ISR 中的副本集合是否需要收缩。这里的收缩是指，把ISR副本集合中那些与Leader差距过大的副本移除的过程。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps127.jpg) 

相对的，有收缩，就会有扩大，在Follower抓取数据时，判断副本状态，满足扩大ISR条件后，就可以提交分区变更请求。完成ISR列表的变更。

向集群Broker传播ISR的变更。ISR发生变化（包含Shrink和Expand）都会执行传播逻辑。ReplicaManager每间隔2500毫秒就会根据条件，将ISR变化的结果传递给集群的其他Broker。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps128.jpg) 

## 2.6 消费消息

数据已经存储到了Kafka的数据文件中，接下来应用程序就可以使用Kafka Consumer API 向Kafka订阅主题，并从订阅的主题上接收消息了。

### 2.6.1 消费消息的基本步骤

（一）建Map类型的配置对象，根据场景增加相应的配置属性：

| 参数名              | 参数作用                                           | 类型 | 默认值 | 推荐值                              |
| ----------------------------- | ------------------------------------------------------------ | -------------- | ---------------- | --------------------------------------------- |
| bootstrap.servers             | 集群地址，格式为：brokerIP1:端口号,brokerIP2:端口号          | 必须           |                  |                                               |
| key.deserializer              | 对数据Key进行反序列化的类完整名称                            | 必须           |                  | Kafka提供的字符串反序列化类：StringSerializer |
| value.deserializer            | 对数据Value进行反序列化的类完整名称                          | 必须           |                  | Kafka提供的字符串反序列化类：ValueSerializer  |
| group.id                      | 消费者组ID，用于标识完整的消费场景，一个组中可以包含多个不同的消费者对象。 | 必须           |                  |                                               |
| auto.offset.reset             |                                                              |                |                  |                                               |
| group.instance.id             | 消费者实例ID，如果指定，那么在消费者组中使用此ID作为memberId前缀 | 可选           |                  |                                               |
| partition.assignment.strategy | 分区分配策略                                                 | 可选           |                  |                                               |
| enable.auto.commit            | 启用偏移量自动提交                                           | 可选           | true             |                                               |
| auto.commit.interval.ms       | 自动提交周期                                                 | 可选           | 5000ms           |                                               |
| fetch.max.bytes               | 消费者获取服务器端一批消息最大的字节数。如果服务器端一批次的数据大于该值（50m）仍然可以拉取回来这批数据，因此，这不是一个绝对最大值。一批次的大小受message.max.bytes （broker config）or max.message.bytes （topic config）影响 | 可选           | 52428800（50 m） |                                               |
| offsets.topic.num.partitions  | 偏移量消费主题分区数                                         | 可选           | 50               |                                               |

（二）创建消费者对象

根据配置创建消费者对象KafkaConsumer，向Kafka订阅（subscribe）主题消息，并向Kafka发送请求（poll）获取数据。

（三）获取数据

Kafka会根据消费者发送的参数，返回数据对象ConsumerRecord。返回的数据对象中包括指定的数据。

| 数据项 | 数据含义 |
| ---------------- | ------------------ |
| topic            | 主题名称           |
| partition        | 分区号             |
| offset           | 偏移量             |
| timestamp        | 数据时间戳         |
| key              | 数据key            |
| value            | 数据value         |

（四）关闭消费者

消费者消费完数据后，需要将对象关闭用以释放资源。一般情况下，消费者无需关闭。

### 2.6.2 消费消息的基本代码

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps130.jpg) 

```
package com.atguigu.test;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.time.Duration;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
public class KafkaConsumerTest {
  public static void main(String[] args) {
 // TODO 创建消费者配置参数集合
  Map<String, Object> paramMap = new HashMap<>();
  paramMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
  paramMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
   paramMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
   paramMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
  // TODO 通过配置，创建消费者对象
 KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(paramMap);
 // TODO 订阅主题
 consumer.subscribe(Collections.singletonList("test"));
 // TODO 消费数据
 final ConsumerRecords<String, String> poll = consumer.poll(Duration.ofMillis(100));
 // TODO 遍历数据
  for (ConsumerRecord<String, String> record : poll) {
    System.out.println( record.value() );
 }
 // TODO 关闭消费者
   consumer.close();
  }
}
```

需要注意：消费者消费数据默认的偏移量是LEO + 1的数据，如果消费消费者在开启之前就有数据了，默认是消费不到的，只能消费到在消费者开启之后的数据。

为了解决这个问题，需要设置消费之的参数

```
    paramMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
```

这个参数加上后能够使得数据从头到尾进行消费。

如果从中间开始消费也是可以的，后面会具体讲解。



### 2.6.3 消费消息的基本原理

从数据处理的角度来讲，消费者和生产者的处理逻辑都相对比较简单。

Producer生产者的基本数据处理逻辑就是向Kafka发送数据，并获取Kafka的数据接收确认响应。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps131.jpg) 

而消费者的基本数据处理逻辑就是向Kafka请求数据，并获取Kafka返回的数据。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps132.jpg) 

逻辑确实很简单，但是Kafka为了能够构建高吞吐，高可靠性，高并发的分布式消息传输系统，所以在很多细节上进行了扩展和改善：比如生产者可以指定分区，可以异步和同步发送数据，可以进行幂等性操作和事务处理。对应的，消费者功能和处理细节也进行了扩展和改善。

#### **2.6.3.1消费者组**

##### **2.6.3.1.1 消费数据的方式：push & pull**

Kafka的主题如果就一个分区的话，那么在硬件配置相同的情况下，消费者Consumer消费主题数据的方式没有什么太大的差别。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps133.jpg) 

不过，Kafka为了能够构建高吞吐，高可靠性，高并发的分布式消息传输系统，它的主题是允许多个分区的，那么就会发现不同的消费数据的方式区别还是很大的。

l 如果数据由Kafka进行推送（push），那么多个分区的数据同时推送给消费者进行处理，明显一个消费者的消费能力是有限的，那么消费者无法快速处理数据，就会导致数据的积压，从而导致网络，存储等资源造成极大的压力，影响吞吐量和数据传输效率。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps134.jpg) 

l 如果kafka的分区数据在内部可以存储的时间更长一些，再由消费者根据自己的消费能力向kafka申请（拉取）数据，那么整个数据处理的通道就会更顺畅一些。Kafka的Consumer就采用的这种拉取数据的方式。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps135.jpg) 

##### 2.6.3.1.2 消费者组Consumer Group

消费者可以根据自身的消费能力主动拉取Kafka的数据，但是毕竟自身的消费能力有限，如果主题分区的数据过多，那么消费的时间就会很长。对于kafka来讲，数据就需要长时间的进行存储，那么对Kafka集群资源的压力就非常大。

如果希望提高消费者的消费能力，并且减少kafka集群的存储资源压力。所以有必要对消费者进行横向伸缩，从而提高消息消费速率。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps136.jpg) 

不过这么做有一个问题，就是每一个消费者是独立，那么一个消费者就不能消费主题中的全部数据，简单来讲，就是对于某一个消费者个体来讲，主题中的部分数据是没有消费到的，也就会认为数据丢了，这个该如何解决呢？那如果我们将这多个消费者当成一个整体，是不是就可以了呢？这就是所谓的消费者组 Consumer Group。**在kafka中，每个消费者都对应一个消费组，消费者可以是一个线程，一个进程，一个服务实例，如果kafka想要消费消息，那么需要指定消费那个topic的消息以及自己的消费组id(groupId)。**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps137.jpg) 

#### **2.6.3.2调度（协调）器Coordinator**

记录着当前分区的消费的位置

消费者想要拉取数据，首先必须要加入到一个组中，成为消费组中的一员，同样道理，如果消费者出现了问题，也应该从消费者组中剥离。而这种加入组和退出组的处理，都应该由专门的管理组件进行处理，这个组件在kafka中，我们称之为消费者组调度器（协调）（Group Coordinator）

Group Coordinator是Broker上的一个组件，用于管理和调度消费者组的成员、状态、分区分配、偏移量等信息。每个Broker都有一个Group Coordinator对象，负责管理多个消费者组，但每个消费者组只有一个Group Coordinator

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps138.jpg) 

#### 2.6.3.3消费者分配策略Assignor

消费者想要拉取主题分区的数据，首先必须要加入到一个组中。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps139.jpg) 

但是一个组中有多个消费者的话，那么每一个消费者该如何消费呢，是不是像图中一样的消费策略呢？如果是的话，那假设消费者组中只有2个消费者或有4个消费者，和分区的数量不匹配，怎么办？所以这里，我们需要给大家介绍一下，Kafka中基本的消费者组中的消费者和分区之间的分配规则：

同一个消费者组的消费者都订阅同一个主题，所以消费者组中的多个消费者可以共同消费一个主题中的所有数据。

**为了避免数据被重复消费，所以主题一个分区的数据只能被组中的一个消费者消费**，也就是说不能两个消费者同时消费一个分区的数据。但是反过来，一个消费者是可以消费多个分区数据的。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps140.jpg) 

消费者组中的消费者数量最好不要超出主题分区的数据，就会导致多出的消费者是无法消费数据的，造成了资源的浪费。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps141.jpg) 

**消费者中的每个消费者到底消费哪一个主题分区，这个分配策略其实是由消费者的Leader决定的，这个Leader我们称之为群主。**群主是多个消费者中，第一个加入组中的消费者，其他消费者我们称之为Follower，称呼上有点类似与分区的Leader和Follower。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps142.jpg) 

当消费者加入群组的时候，会发送一个JoinGroup请求。群主负责给每一个消费者分配分区。

每个消费者只知道自己的分配信息，只有群主知道群组内所有消费者的分配信息。

指定分配策略的基本流程：

(1) 第一个消费者设定group.id为test，向当前负载最小的节点发送请求查找消费调度器

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps143.jpg) 

(2) 找到消费调度器后，消费者向调度器节点发出JOIN_GROUP请求，加入消费者组。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps144.jpg) 

(3) 当前消费者当选为群主后，根据消费者配置中分配策略设计分区分配方案，并将分配好的方案告知调度器

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps145.jpg) 

(4) 此时第二个消费者设定group.id为test，申请加入消费者组

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps146.jpg) 

(5) 加入成功后，kafka将消费者组状态切换到准备rebalance，关闭和消费者的所有链接，等待它们重新加入。客户端重新申请加入，kafka从消费者组中挑选一个作为leader，其它的作为follower。（步骤和之前相同，我们假设还是之前的消费者为Leader）

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps147.jpg) 

(6) Leader会按照分配策略对分区进行重分配，并将方案发送给调度器，由调度器通知所有的成员新的分配方案。组成员会按照新的方案重新消费数据

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps148.jpg) 

Kafka提供的分区分配策略常用的有4个：

**1、RoundRobinAssignor（轮询分配策略）**

每个消费者组中的消费者都会含有一个自动生产的UUID作为memberid。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps149.jpg) 

轮询策略中会将每个消费者按照memberid进行排序，所有member消费的主题分区根据主题名称进行排序。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps150.jpg) 

将主题分区轮询分配给对应的订阅用户，注意未订阅当前轮询主题的消费者会跳过。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps151.jpg) 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps152.jpg) 

从图中可以看出，轮询分配策略是存在缺点的，并不是那么的均衡，如果test1-2分区能够分配给消费者ccc是不是就完美了。

**2、RangeAssignor（范围分配策略）**

按照每个topic的partition数计算出每个消费者应该分配的分区数量，然后分配，分配的原则就是一个主题的分区尽可能的平均分，如果不能平均分，那就按顺序向前补齐即可。

所谓按顺序向前补齐就是：

```
假设【1,2,3,4,5】5个分区分给2个消费者：
5 / 2 = 2 先确定平均消费多少个
5 % 2 = 1  确定前多少个要多分1个
=> 剩余的一个补在第一个中[2+1][2] => 结果为[1,2,3][4,5]

假设【1,2,3,4,5】5个分区分到3个消费者:
5 / 3 = 1, 5 % 3 = 2 => 剩余的两个补在第一个和第二个中[1+1][1+1][1] => 结果为[1,2][3,4][5]
```

![img](file:///C:\Users\wangp\AppData\Local\Temp\ksohtml16168\wps153.jpg) 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps154.jpg) 

缺点：Range分配策略针对单个Topic的情况下显得比较均衡，但是假如Topic多的话, member排序靠前的可能会比member排序靠后的负载多很多。是不是也不够理想。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps155.jpg) 

还有就是如果新增或移除消费者成员，那么会导致每个消费者都需要去建立新的分区节点的连接，更新本地的分区缓存，效率比较低。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps156.jpg) 

**3、StickyAssignor（粘性分区）**

在第一次分配后，每个组成员都保留分配给自己的分区信息。如果有消费者加入或退出，那么在进行分区再分配时（一般情况下，消费者退出45s后，才会进行再分配，因为需要考虑可能又恢复的情况），尽可能保证消费者原有的分区不变，重新对加入或退出消费者的分区进行分配。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps157.jpg) 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps158.jpg) 

从图中可以看出，粘性分区分配策略分配的会更加均匀和高效一些。

**4、CooperativeStickyAssignor**

前面的三种分配策略再进行重分配时使用的是EAGER协议，会让当前的所有消费者放弃当前分区，关闭连接，资源清理，重新加入组和等待分配策略。明显效率是比较低的，所以从Kafka2.4版本开始，在粘性分配策略的基础上，优化了重分配的过程，使用的是COOPERATIVE协议，特点就是在整个再分配的过程中从图中可以看出，粘性分区分配策略分配的会更加均匀和高效一些，COOPERATIVE协议将一次全局重平衡，改成每次小规模重平衡，直至最终收敛平衡的过程。

Kafka消费者默认的分区分配就是RangeAssignor，CooperativeStickyAssignor

#### **2.6.3.4偏移量offset**

偏移量offset是消费者消费数据的一个非常重要的属性。默认情况下，消费者如果不指定消费主题数据的偏移量，那么消费者启动消费时，无论当前主题之前存储了多少历史数据，消费者只能从连接成功后当前主题最新的数据偏移位置读取，而无法读取之前的任何数据，如果想要获取之前的数据，就需要设定配置参数或指定数据偏移量。

##### **2.6.3.4.1 起始偏移量**

在消费者的配置中，我们可以增加偏移量相关参数auto.offset.reset，用于从最开始获取主题数据， 

```
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.time.Duration;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

public class KafkaConsumerTest {

  public static void main(String[] args) {

 

   // TODO 创建消费者配置参数集合
    Map<String, Object> paramMap = new HashMap<>();
    paramMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   paramMap.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

   paramMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

  paramMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
   paramMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
   // TODO 通过配置，创建消费者对象

   KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(paramMap);

   // TODO 订阅主题

   consumer.subscribe(Arrays.asList("test"));
  while ( true ) {
      // TODO 消费数据

     final ConsumerRecords<String, String> poll = consumer.poll(Duration.ofMillis(100));
     // TODO 遍历数据
     for (ConsumerRecord<String, String> record : poll) {
      System.out.println( record );
    }
   }

  }

}
```

参数取值有3个：

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps160.jpg) 

earliest：对于同一个消费者组，从头开始消费。就是说如果这个topic有历史消息存在，现在新启动了一个消费者组，且auto.offset.reset=earliest，那将会从头开始消费（未提交偏移量的场合）。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps161.jpg) 

latest：对于同一个消费者组，消费者只能消费到连接topic后，新产生的数据（未提交偏移量的场合）。

  ![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps162.jpg)

 none：生产环境不使用

##### **2.6.3.4.3 指定偏移量消费**

除了从最开始的偏移量或最后的偏移量读取数据以外，Kafka还支持从指定的偏移量的位置开始消费数据。 

```
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.time.Duration;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class KafkaConsumerOffsetTest {

  public static void main(String[] args) {

 
   Map<String, Object> paramMap = new HashMap<>();

   paramMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

  paramMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

   paramMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

   paramMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
   KafkaConsumer<String, String> c = new KafkaConsumer<String, String>(paramMap);

  // TODO 订阅主题

   c.subscribe(Collections.singletonList("test"));

   // TODO 拉取数据，获取基本集群信息

   c.poll(Duration.ofMillis(100));
   // TODO 根据集群的基本信息配置需要消费的主题及偏移量

  final Set<TopicPartition> assignment = c.assignment();
   for (TopicPartition topicPartition : assignment) {

    if ( topicPartition.topic().equals("test") ) {

      c.seek(topicPartition, 0);

   }

  }

  // TODO 拉取数据

   while (true) {

     final ConsumerRecords<String, String> poll = c.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : poll) {

      System.out.println( record.value() );

     }

   }

  }

}
```



##### **2.6.3.4.4 偏移量提交**

生产环境中，消费者可能因为某些原因或故障重新启动消费，那么如果不知道之前消费数据的位置，重启后再消费，就可能重复消费（earliest）或漏消费（latest）。所以Kafka提供了保存消费者偏移量的功能，而这个功能需要由消费者进行提交操作。这样消费者重启后就可以根据之前提交的偏移量进行消费了。注意，一旦消费者提交了偏移量，那么kafka会优先使用提交的偏移量进行消费。此时，auto.offset.reset参数是不起作用的。

 自动提交

所谓的自动提交就是消费者消费完数据后，无需告知kafka当前消费数据的偏移量，而是由消费者客户端API周期性地将消费的偏移量提交到Kafka中。这个周期默认为5000ms，可以通过配置进行修改。

 

```
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.time.Duration;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;


public class KafkaConsumerCommitAutoTest {

  public static void main(String[] args) {
    // TODO 创建消费者配置参数集合
   Map<String, Object> paramMap = new HashMap<>();
   paramMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
  // TODO 启用自动提交消费偏移量，默认取值为true
   paramMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
   // TODO 设置自动提交offset的时间周期为1000ms，默认5000ms
   paramMap.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 1000);
   paramMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
  paramMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
   paramMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
   // TODO 通过配置，创建消费者对象
   KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(paramMap);
  // TODO 订阅主题
    consumer.subscribe(Arrays.asList("test"));
    while ( true ) {
     // TODO 消费数据

     final ConsumerRecords<String, String> poll = consumer.poll(Duration.ofMillis(100));

     // TODO 遍历数据
     for (ConsumerRecord<String, String> record : poll) {
       System.out.println( record );
     }
   }

  }

}
```

手动提交

基于时间周期的偏移量提交，是我们无法控制的，一旦参数设置的不合理，或单位时间内数据量消费的很多，却没有来及的自动提交，那么数据就会重复消费。所以Kafka也支持消费偏移量的手动提交，也就是说当消费者消费完数据后，自行通过API进行提交。不过为了考虑效率和安全，kafka同时提供了异步提交和同步提交两种方式供我们选择。注意：需要禁用自动提交auto.offset.reset=false，才能开启手动提交

异步提交：向Kafka发送偏移量offset提交请求后，就可以直接消费下一批数据，因为无需等待kafka的提交确认，所以无法知道当前的偏移量一定提交成功，所以安全性比较低，但相对，消费性能会提高

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps165.jpg) 



```
import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.clients.consumer.ConsumerRecord;

import org.apache.kafka.clients.consumer.ConsumerRecords;

import org.apache.kafka.clients.consumer.KafkaConsumer;

import org.apache.kafka.common.serialization.StringDeserializer;

 

import java.time.Duration;

import java.util.Arrays;

import java.util.HashMap;

import java.util.Map;

 

public class KafkaConsumerCommitASyncTest {

  public static void main(String[] args) {

 
   // TODO 创建消费者配置参数集合

   Map<String, Object> paramMap = new HashMap<>();

   paramMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   // TODO 禁用自动提交消费偏移量，默认取值为true

   paramMap.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

   paramMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

   paramMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

   paramMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

   // TODO 通过配置，创建消费者对象

   KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(paramMap);

   // TODO 订阅主题

   consumer.subscribe(Arrays.asList("test"));

   while ( true ) {

     // TODO 消费数据

     final ConsumerRecords<String, String> poll = consumer.poll(Duration.ofMillis(100));

     // TODO 遍历处理数据

     for (ConsumerRecord<String, String> record : poll) {

       System.out.println( record );

    }

     // TODO 异步提交偏移量

     //   此处需要注意，需要在拉取数据完成处理后再提交

     //   否则提前提交了，但数据处理失败，下一次消费数据就拉取不到了

     consumer.commitAsync();

   }

  }

}
```

同步提交：必须等待Kafka完成offset提交请求的响应后，才可以消费下一批数据，一旦提交失败，会进行重试处理，尽可能保证偏移量提交成功，但是依然可能因为以外情况导致提交请求失败。此种方式消费效率比较低，但是安全性高。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps166.jpg) 



```
import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.clients.consumer.ConsumerRecord;

import org.apache.kafka.clients.consumer.ConsumerRecords;

import org.apache.kafka.clients.consumer.KafkaConsumer;

import org.apache.kafka.common.serialization.StringDeserializer;

 

import java.time.Duration;

import java.util.Arrays;

import java.util.HashMap;

import java.util.Map;

 

public class KafkaConsumerCommitSyncTest {

  public static void main(String[] args) {

 
   // TODO 创建消费者配置参数集合

   Map<String, Object> paramMap = new HashMap<>();

   paramMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

   // TODO 禁用自动提交消费偏移量，默认取值为true

   paramMap.put(ConsumerConfiENABLE_AUTO_COMMIT_CONFIG, false);

   paramMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

   paramMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

    paramMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

   // TODO 通过配置，创建消费者对象

   KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(paramMap);

   // TODO 订阅主题

   consumer.subscribe(Arrays.asList("test"));

   while ( true ) {

     // TODO 消费数据

     final ConsumerRecords<String, String> poll = consumer.poll(Duration.ofMillis(100));

     // TODO 遍历处理数据

     for (ConsumerRecord<String, String> record : poll) {

       System.out.println( record );

     }

     // TODO 同步提交偏移量

     //   此处需要注意，需要在拉取数据完成处理后再提交

     //   否则提前提交了，但数据处理失败，下一次消费数据就拉取不到了

     consumer.commitSync();

   }

  }

}
```



#### **2.6.3.5消费者事务**

无论偏移量使用自动提交还是，手动提交，特殊场景中数据都有可能会出现重复消费。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps167.jpg) 

如果提前提交偏移量，再处理业务，又可能出现数据丢失的情况。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps168.jpg) 

对于单独的Consumer来讲，事务保证会比较弱，尤其是无法保证提交的信息被精确消费，主要原因就是消费者可以通过偏移量访问信息，而不同的数据文件生命周期不同，同一事务的信息可能会因为重启导致被删除的情况。所以一般情况下，想要完成kafka消费者端的事务处理，需要将数据消费过程和偏移量提交过程进行原子性绑定，也就是说数据处理完了，必须要保证偏移量正确提交，才可以做下一步的操作，如果偏移量提交失败，那么数据就恢复成处理之前的效果。

对于生产者事务而言，消费者消费的数据也会受到限制。默认情况下，消费者只能消费到生产者提交的数据，也就是未提交完成的数据，消费者是看不到的。如果想要消费到未提交的数据，需要更高消费事务隔离级别 1

```
import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.clients.consumer.ConsumerRecord;

import org.apache.kafka.clients.consumer.ConsumerRecords;

import org.apache.kafka.clients.consumer.KafkaConsumer;

import org.apache.kafka.common.TopicPartition;

import org.apache.kafka.common.serialization.StringDeserializer;

 

import java.time.Duration;

import java.util.*;

 

public class KafkaConsumerTransactionTest {

  public static void main(String[] args) {
    Map<String, Object> paramMap = new HashMap<>();
    paramMap.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    paramMap.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
   paramMap.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

   // TODO 隔离级别：已提交读，读取已经提交事务成功的数据（默认）
    //paramMap.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");

   // TODO 隔离级别：未提交读，读取已经提交事务成功和未提交事务成功的数据
   paramMap.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_uncommitted");

   paramMap.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
   KafkaConsumer<String, String> c = new KafkaConsumer<String, String>(paramMap);
   c.subscribe(Collections.singletonList("test"));
   while (true) {
     final ConsumerRecords<String, String> poll = c.poll(Duration.ofMillis(100));
     for (ConsumerRecord<String, String> record : poll) {
       System.out.println( record.value() );
     }
   }
  }

}
```



#### **2.6.3.6偏移量的保存**

由于消费者在消费消息的时候可能会由于各种原因而断开消费，当重新启动消费者时我们需要让它接着上次消费的位置offset继续消费，因此消费者需要实时的记录自己以及消费的位置。

0.90版本之前，这个信息是记录在zookeeper内的，在0.90之后的版本，offset保存在__consumer_offsets这个topic内。

每个consumer会定期将自己消费分区的offset提交给kafka内部topic：__consumer_offsets，提交过去的时候，key是consumerGroupId+topic+分区号

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps170.jpg) 

value就是当前offset的值，kafka会定期清理topic里的消息，最后就保留最新的那条数据。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps171.jpg) 

因为__consumer_offsets可能会接收高并发的请求，kafka默认给其分配50个分区(可以通过offsets.topic.num.partitions设置)，均匀分配到Kafka集群的多个Broker中。Kafka采用hash(consumerGroupId) % __consumer_offsets主题的分区数来计算我们的偏移量提交到哪一个分区。因为偏移量也是保存到主题中的，所以保存的过程和生产者生产数据的过程基本相同。

#### **2.6.3.7消费数据**

消费者消费数据时，一般情况下，只是设定了订阅的主题名称，那是如何消费到数据的呢。我们这里说一下服务端拉取数据的基本流程。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka/wps172.jpg) 

(1) 服务端获取到用户拉取数据的请求

Kafka消费客户端会向Broker发送拉取数据的请求FetchRequest，服务端Broker获取到请求后根据请求标记FETCH交给应用处理接口KafkaApis进行处理。

(2) 通过副本管理器拉取数据

副本管理器需要确定当前拉取数据的分区，然后进行数据的读取操作

(3) 判定首选副本

2.4版本前，数据读写的分区都是Leader分区，从2.4版本后，kafka支持Follower副本进行读取。主要原因就是跨机房或者说跨数据中心的场景，为了节约流量资源，可以从当前机房或数据中心的副本中获取数据。这个副本称之未首选副本。

(4) 拉取分区数据

Kafka的底层读取数据是采用日志段LogSegment对象进行操作的。

(5) 零拷贝

为了提高数据读取效率，Kafka的底层采用nio提供的FileChannel零拷贝技术，直接从操作系统内核中进行数据传输，提高数据拉取的效率。
