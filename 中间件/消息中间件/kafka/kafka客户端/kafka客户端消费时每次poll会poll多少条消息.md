在Kafka客户端中，每次调用 `poll()` 方法时，消费的消息数量由多个因素决定，主要包括以下几个参数：

1. **`max.poll.records`**：
   - 这是最直接的配置项，用于指定每次 `poll()` 可以返回的最大消息数。默认值是 `500` 条消息。如果你想控制每次 `poll()` 消费的消息数量，可以调整这个参数。
2. **`fetch.min.bytes`** 和 **`fetch.max.bytes`**：
   - 这些参数用于控制从服务器拉取消息时的最小和最大字节数。如果设置了 `fetch.min.bytes`，客户端会等待直到有足够的字节数的消息可供拉取才会返回，这可能会影响 `poll()` 的行为。`fetch.max.bytes` 则限制了每次拉取的最大数据量，影响了 `poll()` 一次最多可以拉取的消息体积。
3. **`fetch.max.wait.ms`**：
   - 如果消息不够 `fetch.min.bytes`，`fetch.max.wait.ms` 是客户端等待消息到达的最长时间。如果时间到了而消息不够，`poll()` 会返回当前已有的消息。
4. **分区数量和消息量**：
   - 如果消费者订阅了多个分区，`poll()` 可能会从每个分区中拉取少量消息，总量不会超过 `max.poll.records`。因此实际拉取的消息数量取决于分区的数量、每个分区的消息量以及 `max.poll.records` 的设置。

总结来说，每次 `poll()` 消费的消息数量主要由 `max.poll.records` 参数决定，但也受到其他配置项和分区消息量的影响。如果你需要精确控制每次 `poll()` 拉取的消息数量，建议调整 `max.poll.records` 的值。

在Spring Boot 应用中使用 Kafka 时，你可以通过配置文件或者在代码中配置 `max.poll.records` 参数。以下是两种常见的设置方法：

### 1. 在 `application.properties` 或 `application.yml` 中配置

如果你使用的是 `application.properties` 文件：

```
spring.kafka.consumer.max-poll-records=100
```

### 2. 在 Java 代码中配置

如果你更喜欢在 Java 代码中进行配置，可以通过设置 `KafkaConsumer` 的属性来实现：

```java
import org.springframework.boot.autoconfigure.kafka.ConcurrentKafkaListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;

@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory(
            ConcurrentKafkaListenerContainerFactoryConfigurer configurer,
            ConsumerFactory<String, String> kafkaConsumerFactory) {
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        configurer.configure(factory, kafkaConsumerFactory);
        
        // Set the max.poll.records property
        factory.getContainerProperties().setPollTimeout(3000);
        factory.getContainerProperties().setMaxPollRecords(100);

        return factory;
    }
}
```

