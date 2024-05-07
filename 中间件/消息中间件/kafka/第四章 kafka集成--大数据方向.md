# **第1章** Kafka集成

## **4.1** **大数据应用场景**

### **4.1.1** **Flume集成**

Flume也是日志采集器，类似于ELK中的LogStash软件功能。早期设计的功能就是通过Flume采集过来数据，然后将数据写入HDFS分布式文件存储系统，不过，随着功能的扩展，现在也可以把采集的数据写入到kafka当中，作为实时数据使用。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps42.jpg) 

#### **4.1.1.1安装Flume**

##### **4.1.1.1.1安装地址**

Flume官网地址：http://flume.apache.org/

文档查看地址：http://flume.apache.org/FlumeUserGuide.html

下载地址：http://archive.apache.org/dist/flume/

##### **4.1.1.1.2安装部署**

1) 将压缩包apache-flume-1.10.1-bin.tar.gz上传到linux系统的/opt/software目录下
2) 将软件压缩包解压缩到/opt/module目录中，并修改名称

\# 解压缩文件

tar -zxf /opt/software/apache-flume-1.10.1-bin.tar.gz -C /opt/module/

\# 修改名称

mv /opt/module/apache-flume-1.10.1-bin /opt/module/flume

3) 生产环境中，可以设置flume的堆内存为4G或以上。

修改/opt/module/flume/conf/flume-env.sh文件，配置如下参数（虚拟机环境暂不配置）

\# 修改JVM配置

export JAVA_OPTS="-Xms4096m -Xmx4096m -Dcom.sun.management.jmxremote"

#### **4.1.1.2 增加集成配置**

##### **4.1.1.2.1 flume采集数据到Kafka的配置**

1) 在linux系统解压缩后的flume软件目录中，创建job目录

\# 进入flume软件目录

cd /opt/module/flume

\# 创建job目录

mkdir job 

2) 创建配置文件：file_to_kafka.conf

\# 进入job目录 

cd /opt/module/flume/job

\# 创建文件

vim file_to_kafka.conf

3) 增加文件内容

\# 定义组件

a1.sources = r1

a1.channels = c1

 

\# 配置source

a1.sources.r1.type = TAILDIR

a1.sources.r1.filegroups = f1

\# 日志（数据）文件

a1.sources.r1.filegroups.f1 = /opt/module/data/test.log

a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json

 

\# 配置channel

\# 采用Kafka Channel，省去了Sink，提高了效率

a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel

a1.channels.c1.kafka.bootstrap.servers = kafka-broker1:9092,kafka-broker2:9092,kafka-broker3:9092

a1.channels.c1.kafka.topic = test

a1.channels.c1.parseAsFlumeEvent = false

 

\# 组装 

a1.sources.r1.channels = c1

### **4.1.1.3 集成测试**

##### **4.1.1.3.1 启动Zookeeper、Kafka集群**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps43.jpg) 

##### **4.1.1.3.2 执行flume操作采集数据到Kafka**

\# 进入flume

cd /opt/module/flume

\# 执行

bin/flume-ng agent -n a1 -c conf/ -f job/file_to_kafka.conf

### **4.1.2 SparkStreaming集成**

Spark是分布式计算引擎，是一款非常强大的离线分布式计算框架，其中的SparkStreaming模块用于准实时数据处理，其中就可以将Kafka作为数据源进行处理。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps44.jpg) 

#### **4.1.2.1 编写功能代码**

##### **4.1.2.1.1 修改pom.xml文件，增加依赖**

<dependency>

​	<groupId>org.apache.spark</groupId>

​	<artifactId>spark-core_2.12</artifactId>

​	<version>3.3.1</version>

</dependency>

<dependency>

​	<groupId>org.apache.spark</groupId>

​	<artifactId>spark-streaming_2.12</artifactId>

​	<version>3.3.1</version>

</dependency>

<dependency>

​	<groupId>org.apache.spark</groupId>

​	<artifactId>spark-streaming-kafka-0-10_2.12</artifactId>

​	<version>3.3.1</version>

</dependency>

##### **4.1.2.1.2 编写功能代码**

package com.atguigu.kafka.test;

 

import org.apache.kafka.clients.consumer.ConsumerConfig;

import org.apache.kafka.clients.consumer.ConsumerRecord;

import org.apache.spark.SparkConf;

import org.apache.spark.api.java.function.Function;

import org.apache.spark.storage.StorageLevel;

import org.apache.spark.streaming.Duration;

import org.apache.spark.streaming.api.java.JavaInputDStream;

import org.apache.spark.streaming.api.java.JavaPairDStream;

import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;

import org.apache.spark.streaming.api.java.JavaStreamingContext;

import org.apache.spark.streaming.kafka010.ConsumerStrategies;

import org.apache.spark.streaming.kafka010.KafkaUtils;

import org.apache.spark.streaming.kafka010.LocationStrategies;

import scala.Tuple2;

 

import java.util.*;

 

public class Kafka4SparkStreamingTest {

  public static void main(String[] args) throws Exception {

 

​    // TODO 创建配置对象

​    SparkConf conf = new SparkConf();

​    conf.setMaster("local[*]");

​    conf.setAppName("SparkStreaming");

 

​    // TODO 创建环境对象

​    JavaStreamingContext ssc = new JavaStreamingContext(conf, new Duration(3 * 1000));

 

​    // TODO 使用kafka作为数据源

 

​    // 创建配置参数

​    HashMap<String, Object> map = new HashMap<>();

​    map.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");

​    map.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

​    map.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

​    map.put(ConsumerConfig.GROUP_ID_CONFIG,"atguigu");

​    map.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"latest");

 

​    // 需要消费的主题

​    ArrayList<String> strings = new ArrayList<>();

​    strings.add("test");

 

​    JavaInputDStream<ConsumerRecord<String, String>> directStream =

​        KafkaUtils.createDirectStream(

​            ssc,

​            LocationStrategies.PreferBrokers(),

​            ConsumerStrategies.<String, String>Subscribe(strings,map));

 

​    directStream.map(new Function<ConsumerRecord<String, String>, String>() {

​      @Override

​      public String call(ConsumerRecord<String, String> v1) throws Exception {

​        return v1.value();

​      }

​    }).print(100);

 

​    ssc.start();

​    ssc.awaitTermination();

  }

}

#### **4.1.2.2 集成测试**

##### **4.1.2.2.1 启动Zookeeper、Kafka集群**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps45.jpg) 

##### **4.1.2.2.2 执行Spark程序**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps46.jpg) 

### **4.1.3 Flink集成**

Flink是分布式计算引擎，是一款非常强大的实时分布式计算框架，可以将Kafka作为数据源进行处理。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps47.jpg) 

#### **4.1.3.1 编写功能代码**

##### **4.1.3.1.1 修改pom.xml文件，增加相关依赖**

<dependency>

​	<groupId>org.apache.flink</groupId>

​	<artifactId>flink-java</artifactId>

​	<version>1.17.0</version>

</dependency>

<dependency>

​	<groupId>org.apache.flink</groupId>

​	<artifactId>flink-streaming-java</artifactId>

​	<version>1.17.0</version>

</dependency>

<dependency>

​	<groupId>org.apache.flink</groupId>

​	<artifactId>flink-clients</artifactId>

​	<version>1.17.0</version>

</dependency>

<dependency>

​	<groupId>org.apache.flink</groupId>

​	<artifactId>flink-connector-kafka</artifactId>

​	<version>1.17.0</version>

</dependency>

##### **4.1.3.1.2 编写功能代码**

package com.atguigu.kafka;

 

import org.apache.flink.api.common.eventtime.WatermarkStrategy;

import org.apache.flink.api.common.serialization.SimpleStringSchema;

import org.apache.flink.connector.kafka.source.KafkaSource;

import org.apache.flink.connector.kafka.source.enumerator.initializer.OffsetsInitializer;

import org.apache.flink.streaming.api.datastream.DataStreamSource;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

 

public class Kafka4FlinkTest {

  public static void main(String[] args) throws Exception {

 

​    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

 

​    KafkaSource<String> kafkaSource = KafkaSource.<String>builder()

​        .setBootstrapServers("localhost:9092")

​        .setTopics("test")

​        .setGroupId("atguigu")

​        .setStartingOffsets(OffsetsInitializer.latest())

​        .setValueOnlyDeserializer(new SimpleStringSchema())

​        .build();

 

​    DataStreamSource<String> stream = env.fromSource(kafkaSource, WatermarkStrategy.noWatermarks(), "kafka-source");

 

​    stream.print("Kafka");

 

​    env.execute();

  }

}

#### **4.1.3.2 集成测试**

##### **4.1.3.2.1 启动Zookeeper、Kafka集群**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps48.jpg) 

##### **4.1.3.2.2 执行Flink程序**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps49.jpg) 

## **4.2** **Java应用场景**

### **4.2.1** **SpringBoot集成**

Spring Boot帮助您创建可以运行的、独立的、生产级的基于Spring的应用程序。您可以使用Spring Boot创建Java应用程序，这些应用程序可以通过使用java-jar或更传统的war部署启动。

我们的目标是：

Ø 为所有Spring开发提供从根本上更快、广泛访问的入门体验。

Ø 开箱即用，但随着要求开始偏离默认值，请迅速离开。

Ø 提供大型项目（如嵌入式服务器、安全性、指标、健康检查和外部化配置）常见的一系列非功能性功能。

Ø 绝对没有代码生成（当不针对原生图像时），也不需要XML配置。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps50.jpg) 

#### **4.2.1.1 创建SpringBoot项目**

##### **4.2.1.1.1** **创建SpringBoot项目**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps51.jpg) 

##### **4.2.1.1.2 修改pom.xml文件**

<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

​     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <parent>

​    <groupId>org.springframework.boot</groupId>

​    <artifactId>spring-boot-starter-parent</artifactId>

​    <version>3.0.5</version>

​    <relativePath/> <!-- lookup parent from repository -->

  </parent>

  <groupId>com.atguigu</groupId>

  <artifactId>springboot-kafka</artifactId>

  <version>0.0.1</version>

  <name>springboot-kafka</name>

  <description>Kafka project for Spring Boot</description>

  <properties>

​    <java.version>17</java.version>

  </properties>

  <dependencies>

​    <dependency>

​      <groupId>org.springframework.boot</groupId>

​      <artifactId>spring-boot-starter</artifactId>

​    </dependency>

​    <dependency>

​      <groupId>org.springframework.boot</groupId>

​      <artifactId>spring-boot-starter-web</artifactId>

​      <exclusions>

​        <exclusion>

​          <groupId>org.springframework.boot</groupId>

​          <artifactId>spring-boot-starter-logging</artifactId>

​        </exclusion>

​      </exclusions>

​    </dependency>

​    <dependency>

​      <groupId>org.springframework.boot</groupId>

​      <artifactId>spring-boot-starter-test</artifactId>

​      <scope>test</scope>

​    </dependency>

​    <dependency>

​      <groupId>org.springframework.kafka</groupId>

​      <artifactId>spring-kafka</artifactId>

​    </dependency>

​    <dependency>

​      <groupId>org.apache.kafka</groupId>

​      <artifactId>kafka-clients</artifactId>

​      <version>3.6.1</version>

​    </dependency>

​    <dependency>

​      <groupId>com.alibaba</groupId>

​      <artifactId>fastjson</artifactId>

​      <version>1.2.83</version>

​    </dependency>

​    <dependency>

​      <groupId>cn.hutool</groupId>

​      <artifactId>hutool-json</artifactId>

​      <version>5.8.11</version>

​    </dependency>

​    <dependency>

​      <groupId>cn.hutool</groupId>

​      <artifactId>hutool-db</artifactId>

​      <version>5.8.11</version>

​    </dependency>

​    <dependency>

​      <groupId>org.projectlombok</groupId>

​      <artifactId>lombok</artifactId>

​    </dependency>

  </dependencies>

 

  <build>

​    <plugins>

​      <plugin>

​        <groupId>org.springframework.boot</groupId>

​        <artifactId>spring-boot-maven-plugin</artifactId>

​      </plugin>

​    </plugins>

  </build>

</project>

##### **4.2.1.1.3 在resources中增加application.yml文件**

spring:

 kafka:

  bootstrap-servers: localhost:9092

  producer:

   acks: all

   batch-size: 16384

   buffer-memory: 33554432

   key-serializer: org.apache.kafka.common.serialization.StringSerializer

   value-serializer: org.apache.kafka.common.serialization.StringSerializer

   retries: 0

  consumer:

   group-id: test#消费者组

   \#消费方式: 在有提交记录的时候，earliest与latest是一样的，从提交记录的下一条开始消费

   \# earliest：无提交记录，从头开始消费

   \#latest：无提交记录，从最新的消息的下一条开始消费

   auto-offset-reset: earliest

   enable-auto-commit: true #是否自动提交偏移量offset

   auto-commit-interval: 1s #前提是 enable-auto-commit=true。自动提交的频率

   key-deserializer: org.apache.kafka.common.serialization.StringDeserializer

   value-deserializer: org.apache.kafka.common.serialization.StringDeserializer

   max-poll-records: 2

   properties:

​    \#如果在这个时间内没有收到心跳，该消费者会被踢出组并触发{组再平衡 rebalance}

​    session.timeout.ms: 120000

​    \#最大消费时间。此决定了获取消息后提交偏移量的最大时间，超过设定的时间（默认5分钟），服务端也会认为该消费者失效。踢出并再平衡

​    max.poll.interval.ms: 300000

​    \#配置控制客户端等待请求响应的最长时间。

​    \#如果在超时之前没有收到响应，客户端将在必要时重新发送请求，

​    \#或者如果重试次数用尽，则请求失败。

​    request.timeout.ms: 60000

​    \#订阅或分配主题时，允许自动创建主题。0.11之前，必须设置false

​    allow.auto.create.topics: true

​    \#poll方法向协调器发送心跳的频率，为session.timeout.ms的三分之一

​    heartbeat.interval.ms: 40000

​    \#每个分区里返回的记录最多不超max.partitions.fetch.bytes 指定的字节

​    \#0.10.1版本后 如果 fetch 的第一个非空分区中的第一条消息大于这个限制

​    \#仍然会返回该消息，以确保消费者可以进行

​    \#max.partition.fetch.bytes=1048576  #1M

  listener:

   \#当enable.auto.commit的值设置为false时，该值会生效；为true时不会生效

   \#manual_immediate:需要手动调用Acknowledgment.acknowledge()后立即提交

   \#ack-mode: manual_immediate

   missing-topics-fatal: true #如果至少有一个topic不存在，true启动失败。false忽略

   \#type: single #单条消费？批量消费？ #批量消费需要配合 consumer.max-poll-records

   type: batch

   concurrency: 2 #配置多少，就为为每个消费者实例创建多少个线程。多出分区的线程空闲

  template:

   default-topic: "test"

server:

 port: 9999

#### **4.2.1.2 编写功能代码**

##### **4.2.1.2.1** **创建配置类：SpringBootKafkaConfig**

package com.atguigu.springkafka.config;

 

public class SpringBootKafkaConfig {

  public static final String TOPIC_TEST = "test";

  public static final String GROUP_ID = "test";

}

##### **4.2.1.2.2** **创建生产者控制器：KafkaProducerController**

package com.atguigu.springkafka.controller;

 

import com.atguigu.springkafka.config.SpringBootKafkaConfig;

import lombok.extern.slf4j.Slf4j;

import cn.hutool.json.JSONUtil;

import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.kafka.core.KafkaTemplate;

import org.springframework.kafka.support.SendResult;

import org.springframework.web.bind.annotation.*;

 

import org.springframework.util.concurrent.ListenableFuture;

import org.springframework.util.concurrent.ListenableFutureCallback;

 

@RestController

@RequestMapping("/kafka")

@Slf4j

public class KafkaProducerController {

 

 

  @Autowired

  private KafkaTemplate<String, String> kafkaTemplate;

 

  @ResponseBody

  @PostMapping(value = "/produce", produces = "application/json")

  public String produce(@RequestBody Object obj) {

 

​    try {

​      String obj2String = JSONUtil.toJsonStr(obj);

​      kafkaTemplate.send(SpringBootKafkaConfig.TOPIC_TEST, obj2String);

​      return "success";

​    } catch (Exception e) {

​      e.getMessage();

​    }

​    return "success";

  }

}

##### **4.2.1.2.3 创建消费者：KafkaDataConsumer**

package com.atguigu.springkafka.component;

 

import cn.hutool.json.JSONObject;

import cn.hutool.json.JSONUtil;

import lombok.extern.slf4j.Slf4j;

import com.atguigu.springkafka.config.SpringBootKafkaConfig;

import org.springframework.kafka.annotation.KafkaListener;

import org.springframework.kafka.support.KafkaHeaders;

import org.springframework.messaging.handler.annotation.Header;

import org.springframework.stereotype.Component;

 

import java.util.List;

import java.util.Optional;

 

 

@Component

@Slf4j

public class KafkaDataConsumer {

  @KafkaListener(topics = SpringBootKafkaConfig.TOPIC_TEST, groupId = SpringBootKafkaConfig.GROUP_ID)

  public void topic_test(List<String> messages, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {

​    for (String message : messages) {

​      final JSONObject entries = JSONUtil.parseObj(message);

​      System.out.println(SpringBootKafkaConfig.GROUP_ID + " 消费了： Topic:" + topic + ",Message:" + entries.getStr("data"));

​      //ack.acknowledge();

​    }

  }

}

#### **4.2.1.3 集成测试**

##### **4.2.1.3.1 启动ZooKeeper**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps52.jpg) 

##### **4.2.1.3.2** **启动Kafka**

##### **4.2.1.3.3** **启动应用程序**

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps53.jpg) 

##### **4.2.1.3.4** **生产数据测试**

可以采用任何的工具进行测试，我们这里采用postman发送POST数据

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps54.jpg) 

消费者消费数据

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/kafka3/wps55.jpg) 