# RocketMQ
在RocketMQ中有NameServer、Broker、生产者、消费者四种角色。而生产者和消费者实际上就是业务系统，所以这里不需要搭建，真正要搭建的就是NameServer和Broker  
## nameServer安装启动：
 下载安装  
 https://archive.apache.org/dist/rocketmq
 安装：  
 unzip rocketmq-all-4.7.1-bin-release.zip  
 cd rocketmq-all-4.7.1-bin-release  
可以修改一下参数（ vim runserver.sh），默认设置的队栈内存太大了  

```
server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m
```
修改为  
```
server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=32m -XX:MaxMetaspaceSize=50m
```
启动  
```
nohup sh bin/mqnamesrv &
```
启动日志在这里  
```
~/logs/rocketmqlogs/namesrv.log
```

## 启动Broker
修改jvm参数  （runbroker.sh）

```
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
```
修改为  
```
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m"
```
修改配置文件
```
vi conf/broker.conf
```
追加  
```
namesrvAddr = localhost:9876
brokerIP1 = 10.235.5.57
brokerIP2 = 10.235.5.57
```
启动  
```
cd /home/rocketMQ/rocketmq-all-4.7.1-bin-release/bin
nohup sh mqbroker -c ../conf/broker.conf  &
```
-c 参数就是指定配置文件  

日志在这里  
```
tail -f ~/logs/rocketmqlogs/broker.log
```
## 建立demo测试
### 引入依赖
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>SpringCloudParent</artifactId>
        <groupId>com.example</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>rocketmq1</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>4.7.1</version>
        </dependency>


    </dependencies>
</project>
```
### 生产者
```
package product;

import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;

public class Productor1 {
    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, MQBrokerException, RemotingException, InterruptedException {
        //1、创建一个生产者，指定生产组
        DefaultMQProducer product = new DefaultMQProducer("productGroup");
        //2、指定NameServer的地址
        product.setNamesrvAddr("10.235.5.57:9876");
        //3、设置发送超时时间
        product.setSendMsgTimeout(60000);
        //4、启动生产者
        product.start();
        //5、创建一条消息
        Message message = new Message("productGroup", "TagA", "第二条消息".getBytes(RemotingHelper.DEFAULT_CHARSET));
        //6、发送消息并且返回结果
        SendResult send = product.send(message);
        System.out.println(send);
        product.shutdown();
    }
}

```
### 消费者
```
package consumer1;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.MessageExt;

import java.nio.charset.StandardCharsets;
import java.util.List;

public class Consumer1 {
    public static void main(String[] args) {
        try {
            // 创建消费者并设置组名
            DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumerGroup");

            // 指定NameServer的地址
            consumer.setNamesrvAddr("10.235.5.57:9876");

            // 订阅生产者发送消息的主题和标签
            consumer.subscribe("productGroup", "TagA");

            // 注册消息监听器，当有消息时，会回调这个监听器来消费消息
            consumer.registerMessageListener(new MessageListenerConcurrently() {
                @Override
                public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                    for (MessageExt msg : msgs) {
                        System.out.println("Received message: " + new String(msg.getBody(), StandardCharsets.UTF_8));
                    }
                    // 返回消费状态
                    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                }
            });

            // 启动消费者
            consumer.start();

            System.out.println("Consumer started.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

