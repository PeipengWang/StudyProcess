# kafka
## 一、初识kafka
kafka是LinkedIn公司采用Scala语言开发的一个多分区、多副本并且基于Zookeeper协调的分布式消息系统。目前定位为分布式流式处理平台，它以高吞吐、可持久、可水平扩展、支持流处理等多种特性被广泛应用。
<img src="_v_images/20240203141743665_7562.png" alt="" style="width: 799px;">



### 角色：
Producer
生产者即数据发布者，该角色将消息发布到Broker
Zookeper
管理Broker
AR
分区中所有的副本
ISR
所有与Leader部分保持一定程度的副本组成ISR
OSR
与leader副本同步滞后过多的副本



 mv kafka_2.12-3.6.0.tgz kafka-3.6.0-src.tgz apache-zookeeper-3.7.2-bin.tar.gz  kafka/
cd kafka/
tar -zxvf apache-zookeeper-3.7.2-bin.tar.gz 
cd apache-zookeeper-3.7.2-bin/
cd conf/
mkdir data
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg 
./zkServer.sh start
 ./zkServer.sh  status
cd /home/kafka/
cd /home/kafka/apache-zookeeper-3.7.2-bin/bin/
./zkServer.sh stop
vim zookeeper.properties 
cd /tmp/zookeeper/
cd /home/kafka/kafka_2.12-3.6.0/

bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
bin/zookeeper-server-start.sh config/zookeeper.properties


bin/kafka-server-start.sh config/server.properties

nohup sh bin/zookeeper-server-start.sh config/zookeeper.properties > zookeeper.log 2>&1 &
nohup sh bin/kafka-server-start.sh config/server.properties > server.log 2>&1 &

vim connect-standalone.properties 
 
 bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092










