# Apache Camel 基本使用

Apache Camel 是一个基于路由和中介模型的开源集成框架，提供了一套易用的DSL (Domain Specific Language) 来定义路由和处理逻辑。以下是一些常见用法的示例，包括如何使用Camel从Kafka读取消息、处理消息，并将消息发送到其他系统。

### 基本用法示例

#### 1. 从文件读取消息并发送到Kafka

```
java复制代码import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class FileToKafkaExample {
    public static void main(String[] args) throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() {
                from("file:data/input?noop=true")
                    .log("Read file: ${header.CamelFileName}")
                    .to("kafka:your_topic?brokers=localhost:9092");
            }
        });

        context.start();
        Thread.sleep(5000);
        context.stop();
    }
}
```

#### 2. 从Kafka读取消息并写入到文件

```
java复制代码import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class KafkaToFileExample {
    public static void main(String[] args) throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() {
                from("kafka:your_topic?brokers=localhost:9092&groupId=group1")
                    .log("Message received from Kafka: ${body}")
                    .to("file:data/output");
            }
        });

        context.start();
        Thread.sleep(5000);
        context.stop();
    }
}
```

### 高级用法示例

#### 3. 消息过滤和处理

```
java复制代码import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class KafkaFilterExample {
    public static void main(String[] args) throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() {
                from("kafka:your_topic?brokers=localhost:9092&groupId=group1")
                    .filter(body().contains("important"))
                    .log("Important message: ${body}")
                    .to("file:data/important");
            }
        });

        context.start();
        Thread.sleep(5000);
        context.stop();
    }
}
```

#### 4. 消息转换和聚合

```
java复制代码import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class KafkaTransformExample {
    public static void main(String[] args) throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() {
                from("kafka:your_topic?brokers=localhost:9092&groupId=group1")
                    .log("Original message: ${body}")
                    .transform(body().append(" - Processed by Camel"))
                    .to("kafka:processed_topic?brokers=localhost:9092");
            }
        });

        context.start();
        Thread.sleep(5000);
        context.stop();
    }
}
```

#### 5. 使用定时器定期从Kafka读取并处理消息

```
java复制代码import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class KafkaTimerExample {
    public static void main(String[] args) throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() {
                from("timer:kafkaTimer?period=10000")
                    .log("Timer triggered")
                    .to("kafka:your_topic?brokers=localhost:9092&groupId=group1")
                    .log("Message received from Kafka: ${body}")
                    .to("file:data/output");
            }
        });

        context.start();
        Thread.sleep(60000); // 运行1分钟后停止
        context.stop();
    }
}
```

### 6. 处理和转发消息到多个目标系统

```
java复制代码import org.apache.camel.CamelContext;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;

public class KafkaToMultipleTargetsExample {
    public static void main(String[] args) throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() {
                from("kafka:your_topic?brokers=localhost:9092&groupId=group1")
                    .log("Message received from Kafka: ${body}")
                    .multicast()
                    .to("file:data/output", "direct:processInMemory");

                from("direct:processInMemory")
                    .log("Processing in memory: ${body}")
                    .to("kafka:another_topic?brokers=localhost:9092");
            }
        });

        context.start();
        Thread.sleep(5000);
        context.stop();
    }
}
```

通过上述示例，你可以看到Camel强大的集成能力和灵活的DSL用法。你可以根据具体需求，灵活配置路由、处理逻辑、和消息传输方式，以实现复杂的企业集成方案。如果有任何具体需求或问题，可以进一步讨论和调整。