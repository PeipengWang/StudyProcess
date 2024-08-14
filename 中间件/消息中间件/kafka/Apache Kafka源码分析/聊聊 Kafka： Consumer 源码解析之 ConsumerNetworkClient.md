## 一、Consumer 的使用

Consumer 的源码解析主要来看 KafkaConsumer，KafkaConsumer 是 Consumer 接口的实现类。KafkaConsumer 提供了一套封装良好的 API，开发人员可以基于这套 API 轻松实现从 Kafka 服务端拉取消息的功能，这样开发人员根本不用关心与 Kafka 服务端之间网络连接的管理、心跳检测、请求超时重试等底层操作，也不必关心订阅 Topic 的分区数量、分区副本的网络拓扑以及 Consumer Group 的 Rebalance 等 Kafka 具体细节，KafkaConsumer 中还提供了自动提交 offset 的功能，使的开发人员更加关注业务逻辑，提高了开发效率。

## 二、KafkaConsumer 分析

我们先来看下 Consumer 接口，该接口定义了 KafkaConsumer 对外的 API，其核心方法可以分为以下六类：

- **subscribe**() 方法：订阅指定的 Topic，并为消费者自动分配分区。
- **assign**() 方法：用户手动订阅指定的 Topic，并且指定消费的分区，此方法 subscribe() 方法互斥。
- **poll**() 方法：负责从服务端获取消息。
- **commit***() 方法：提交消费者已经消费完成的 offset。
- **seek***() 方法：指定消费者起始消费的位置。
- **pause**()、**resume**() 方法：暂停、继续 Consumer，暂停后 poll() 方法会返回空。

