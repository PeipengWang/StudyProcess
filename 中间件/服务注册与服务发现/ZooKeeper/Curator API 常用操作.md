## 引入依赖

```xml
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>2.9.0</version>
    </dependency>
```

## 建立客户端

```java
 ExponentialBackoffRetry retry = new ExponentialBackoffRetry(3000, 10);
        client = CuratorFrameworkFactory.builder().connectString("")
                .sessionTimeoutMs(60*1000)
                .connectionTimeoutMs(15 * 1000)
                .retryPolicy(retry).namespace("node").build();
```

