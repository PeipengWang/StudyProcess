# kafka
## 一、初识kafka
kafka是LinkedIn公司采用Scala语言开发的一个多分区、多副本并且基于Zookeeper协调的分布式消息系统。目前定位为分布式流式处理平台，它以高吞吐、可持久、可水平扩展、支持流处理等多种特性被广泛应用。
![](_v_images/20240203141743665_7562.png =799x)


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













