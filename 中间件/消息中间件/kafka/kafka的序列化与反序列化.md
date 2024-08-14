# 基本概念

Kafka 的编码与解码主要指消息在生产者发送和消费者接收时的数据序列化与反序列化过程。Kafka 本身支持多种编码/解码方式，通常使用序列化器和反序列化器来实现。

Kafka 的编码与解码机制

在 Kafka 中，消息是以字节数组的形式在 Producer 和 Consumer 之间传递的。因此，数据在发送前需要序列化成字节数组，在接收后需要反序列化成相应的对象。

- **Producer 编码（序列化）**：在 Producer 端，数据在发送前通过序列化器（Serializer）将对象转换成字节数组。
- **Consumer 解码（反序列化）**：在 Consumer 端，接收到的字节数组通过反序列化器（Deserializer）转换回对象。

# 常用的编码方式

## **1、String 编码/解码**

这是最常见的一种方式，将字符串转换为字节数组。

- **Serializer**: `org.apache.kafka.common.serialization.StringSerializer`
- **Deserializer**: `org.apache.kafka.common.serialization.StringDeserializer`

```java
// Producer 配置
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

// Consumer 配置
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

```

## **2、Byte Array 编码/解码**

直接将数据作为字节数组传输。

- **Serializer**: `org.apache.kafka.common.serialization.ByteArraySerializer`
- **Deserializer**: `org.apache.kafka.common.serialization.ByteArrayDeserializer`

```java
// Producer 配置
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, ByteArraySerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, ByteArraySerializer.class.getName());

// Consumer 配置
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, ByteArrayDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ByteArrayDeserializer.class.getName());

```

## **3、Integer 编码/解码**

用于整数类型的编码和解码。

- **Serializer**: `org.apache.kafka.common.serialization.IntegerSerializer`

- **Deserializer**: `org.apache.kafka.common.serialization.IntegerDeserializer`

  ```java
  // Producer 配置
  props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class.getName());
  props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class.getName());
  
  // Consumer 配置
  props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class.getName());
  props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class.getName());
  
  ```

  ## 4、 **JSON 编码/解码**

  将 Java 对象转换为 JSON 字符串，再通过 StringSerializer 和 StringDeserializer 进行传输。

  - **JSON Serializer**: 你可以自己实现，或者使用 `Jackson` 提供的序列化器/反序列化器。

  ```java
  public class JsonSerializer<T> implements Serializer<T> {
      private final ObjectMapper objectMapper = new ObjectMapper();
  
      @Override
      public byte[] serialize(String topic, T data) {
          try {
              return objectMapper.writeValueAsBytes(data);
          } catch (Exception e) {
              throw new SerializationException("Error serializing JSON message", e);
          }
      }
  }
  
  public class JsonDeserializer<T> implements Deserializer<T> {
      private final ObjectMapper objectMapper = new ObjectMapper();
      private final Class<T> tClass;
  
      public JsonDeserializer(Class<T> tClass) {
          this.tClass = tClass;
      }
  
      @Override
      public T deserialize(String topic, byte[] bytes) {
          try {
              return objectMapper.readValue(bytes, tClass);
          } catch (Exception e) {
              throw new SerializationException("Error deserializing JSON message", e);
          }
      }
  }
  
  ```

  

## **4、自定义编码/解码**

你可以根据需求实现自定义的 `Serializer` 和 `Deserializer`。通常是在 `serialize` 和 `deserialize` 方法中编写自定义的编码/解码逻辑。

```java
public class CustomSerializer implements Serializer<CustomObject> {
    @Override
    public byte[] serialize(String topic, CustomObject data) {
        // 自定义序列化逻辑
    }
}

public class CustomDeserializer implements Deserializer<CustomObject> {
    @Override
    public CustomObject deserialize(String topic, byte[] data) {
        // 自定义反序列化逻辑
    }
}

```

