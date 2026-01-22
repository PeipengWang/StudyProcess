例如

```
  from(String.format("rocketmq:%s?consumerGroup=%s&namesrvAddr="+nameServerAddr,
                ROCKETMQ_TOPIC, ROCKETMQ_CONSUMER_GROUP))
```

### 一、核心含义

这段代码的作用是**动态拼接 Camel RocketMQ 消费者端点的 URL**，最终生成一个可连接到指定 RocketMQ 服务的消费者端点，用于从指定 Topic 接收消息。

### 二、逐行拆解（语法 + 含义）

先看完整代码：

```
from(String.format("rocketmq:%s?consumerGroup=%s&namesrvAddr="+nameServerAddr,
        ROCKETMQ_TOPIC, ROCKETMQ_CONSUMER_GROUP))
```

#### 1. 基础语法：String.format () 字符串格式化

`String.format()` 是 Java 用于动态拼接字符串的工具，格式为：

```
String.format("模板字符串", 参数1, 参数2, ...)
```

- 模板中的 `%s` 是「字符串占位符」，会按顺序被后面的参数替换；
- 你代码中**存在语法小问题**：`&namesrvAddr="+nameServerAddr` 应该放到模板里，而非直接拼接（下文会修复）。

#### 2. 模板字符串拆解（关键：RocketMQ 端点格式）

Camel RocketMQ 组件的端点格式为：

```
rocketmq:TOPIC?参数1=值1&参数2=值2&参数3=值3
```

|            模板部分            |                             含义                             |
| :----------------------------: | :----------------------------------------------------------: |
|          `rocketmq:`           |           Camel 识别 RocketMQ 组件的前缀（固定）；           |
|         `%s`（第一个）         |   占位符，会被 `ROCKETMQ_TOPIC`（值为 "demo-topic"）替换；   |
|              `?`               |       分隔符，后面跟端点参数（类似 URL 的查询参数）；        |
|       `consumerGroup=%s`       | 消费者组参数，第二个 `%s` 会被 `ROCKETMQ_CONSUMER_GROUP`（"MY_CAMEL_CONSUMER_GROUP"）替换； |
| `&namesrvAddr=`+nameServerAddr | ❌ 这里是**语法问题**：把 `namesrvAddr` 直接拼接在模板外，正确写法应放到模板内（下文修复）； |

#### 3. 替换后的最终端点（示例）

假设：

- `ROCKETMQ_TOPIC = "demo-topic"`
- `ROCKETMQ_CONSUMER_GROUP = "MY_CAMEL_CONSUMER_GROUP"`
- `nameServerAddr = "127.0.0.1:9876"`

则拼接后生成的端点 URL 为：

```
rocketmq:demo-topic?consumerGroup=MY_CAMEL_CONSUMER_GROUP&namesrvAddr=127.0.0.1:9876
```

这个 URL 告诉 Camel：

- 连接到 `127.0.0.1:9876` 这个 RocketMQ NameServer；
- 订阅 `demo-topic` 这个 Topic；
- 以 `MY_CAMEL_CONSUMER_GROUP` 为消费者组身份消费消息。

### RocketMQ 端点核心参数说明

除了代码中的 3 个核心参数，补充常用参数（方便你扩展）：

|        参数名         | 必填 |                             含义                             |         示例值          |
| :-------------------: | :--: | :----------------------------------------------------------: | :---------------------: |
|    `consumerGroup`    |  是  |     消费者组（RocketMQ 必选，用于负载均衡和消息重试）；      | MY_CAMEL_CONSUMER_GROUP |
|     `namesrvAddr`     |  是  |    NameServer 地址（RocketMQ 服务入口，格式：IP: 端口）；    |     127.0.0.1:9876      |
|    `messageModel`     |  否  | 消费模式：CLUSTERING（集群模式，默认）/BROADCASTING（广播模式）； |       CLUSTERING        |
|  `consumeThreadNums`  |  否  |                   消费线程数（默认 20）；                    |           10            |
| `batchConsumeMaxSize` |  否  |                 批量消费最大条数（默认 1）；                 |           100           |

#### 扩展示例（添加消费模式 + 线程数）

```
from(String.format(
    "rocketmq:%s?consumerGroup=%s&namesrvAddr=%s&messageModel=CLUSTERING&consumeThreadNums=10",
    ROCKETMQ_TOPIC,
    ROCKETMQ_CONSUMER_GROUP,
    nameServerAddr
))
```