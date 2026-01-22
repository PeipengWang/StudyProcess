### 第 1 步：新增 RocketMQ Consumer（替代 Kafka from）

这是**新增类，不是改 Camel**

```
@Service
@RocketMQMessageListener(
        topic = "tm-topic",
        consumerGroup = "tm-consumer-group"
)
public class RocketMQTmConsumer implements RocketMQListener<String> {

    @Resource
    private ProducerTemplate producerTemplate;

    @Override
    public void onMessage(String message) {
        // 只做一件事：交给 Camel
        producerTemplate.sendBody("seda:rocketmqInput", message);
    }
}

```

 **这里等价于 Kafka 的 `from("kafka:xxx")`**

### 第 2 步：在 Camel 中新增“统一 RocketMQ 入口路由”

👉 **这是 KafkaRouter 中最关键的改造点**

🔥 新增一个入口 Route（替代所有 kafka:xxx）

```
from("seda:rocketmqInput?concurrentConsumers=10")
    .routeId("rocketmq-input-route")

    // RocketMQ 没有 kafka.TOPIC，这里要补
    .process(exchange -> {
        // RocketMQ topic 可以通过 Header 传
        // 如果你一个 consumer 对一个 topic，这里可写死
        exchange.getIn().setHeader("mq.TOPIC", "tm-topic");
    })

    .process(new StringParseProcess())

    .multicast().parallelProcessing()
        .to("direct:pick_tm-topic")
        .to("direct:payload_tm-topic")
        .to("direct:alarm_tm-topic")
        .to("direct:parameterAlarm_tm-topic")
        .to("direct:cacheTm_tm-topic")
        .to("direct:alarmP2p_tm-topic")
        .to("direct:multiStatePick_tm-topic")
        .to("direct:commandMonitor_tm-topic");

```

### 第 3 步：统一替换 `kafka.TOPIC` Header（非常关键）

你现有代码里大量使用了：

```
exchange.getIn().getHeader("kafka.TOPIC")
```

🚨 **RocketMQ 是没有这个 Header 的**

✅ 推荐做法：抽象一个 MQ Header

统一使用：

```
exchange.getIn().getHeader("mq.TOPIC")
```

------

示例：修改一个 Processor

❌ 原 Kafka 写法

```
tmPickService.parseKafkaMessage(
    exchange.getIn().getBody(TmResult.class),
    exchange.getIn().getHeader("kafka.TOPIC").toString()
);
```

✅ 改造后（MQ 无关）

```
tmPickService.parseKafkaMessage(
    exchange.getIn().getBody(TmResult.class),
    exchange.getIn().getHeader("mq.TOPIC").toString()
);
```

📌 **一次改造，全链路通用**

### 第 4 步：保留你现有的 direct 路由（几乎不用动）

下面这些 **一行都不用改**：

```
from("direct:pick_".concat(topic))
from("direct:payload_".concat(topic))
from("direct:alarm_".concat(topic))
from("direct:parameterAlarm_".concat(topic))
from("direct:cacheTm_".concat(topic))
...
```

👉 **Camel 的价值就在这里：入口换了，业务不动**

# 你这套代码在 RocketMQ 下的注意事项（非常重要）

## 1️⃣ RocketMQ ACK 与 Camel 异常

你现在很多 Processor：

```
catch (Exception e) {
    e.printStackTrace();
}
```

⚠️ **在 RocketMQ 下必须保证：**

- 异常不要抛回 `onMessage`
- 否则 MQ 会重试 / 死信

### 推荐：统一 Camel 异常兜底

```
onException(Exception.class)
    .handled(true)
    .log("Camel 处理异常: ${exception.message}");
```

------

## 2️⃣ 并发模型对比

| 层         | Kafka              | RocketMQ                 |
| ---------- | ------------------ | ------------------------ |
| 消费并发   | Kafka Consumer     | MQ PushConsumer          |
| Camel 并发 | direct / multicast | seda.concurrentConsumers |
| 解耦       | 一般               | **更强**                 |

📌 你现在的代码 **非常适合 RocketMQ**