下面是一个重构后的精简版 `TmSubscribeController` 类，仅保留了 **chart 模式**，并使用 `BlockingQueue<EpduResultForCK>` 替代原有 `List + ReentrantLock + Condition` 实现：

------

## ✅ 重构重点

- 保留核心逻辑：订阅、取消订阅、接收数据、定时推送
- 删除：list 模式、复杂锁、条件变量
- 使用：`LinkedBlockingQueue<EpduResultForCK>` 实现缓存+阻塞机制

------

## ✅ 重构后的代码（简洁清晰）

```java


import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.DestinationVariable;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Controller;

import java.text.SimpleDateFormat;
import java.util.*;
import java.util.concurrent.*;

@Slf4j
@Controller
public class TmSubscribeController {

    private final SimpMessagingTemplate messagingTemplate;
    private final SourceService sourceService;
    private final ObjectMapper objectMapper;
    private final long cacheTimeMs;

    private final Map<String, Map<SubscribeInfo, BlockingQueue<EpduResultForCK>>> queueMap = new ConcurrentHashMap<>();
    private final Map<String, Map<SubscribeInfo, ScheduledFuture<?>>> scheduleMap = new ConcurrentHashMap<>();
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(4);

    @Autowired
    public TmSubscribeController(SimpMessagingTemplate messagingTemplate,
                                 SourceService sourceService,
                                 ObjectMapper objectMapper) {
        this.messagingTemplate = messagingTemplate;
        this.sourceService = sourceService;
        this.objectMapper = objectMapper;
        this.cacheTimeMs = 100L; // 可替换为 @Value 注入

        // 初始化每个 topic 的映射
        for (String topic : sourceService.getTopics()) {
            queueMap.put(topic, new ConcurrentHashMap<>());
            scheduleMap.put(topic, new ConcurrentHashMap<>());
        }
    }

    @MessageMapping("/search/subscribe/{sourceStr}")
    public void subscribe(@DestinationVariable String sourceStr, SubscribeInfo info) {
        String topic = SourceConfig.source2KafkaTopic(SourceConfig.string2Source(sourceStr));
        queueMap.get(topic).put(info, new LinkedBlockingQueue<>());

        ScheduledFuture<?> future = scheduler.scheduleWithFixedDelay(() -> sendChartData(topic, sourceStr, info),
                cacheTimeMs, cacheTimeMs, TimeUnit.MILLISECONDS);
        scheduleMap.get(topic).put(info, future);

        log.info("用户 {} 订阅了 chart 数据：{}", info.getUid(), info.getParameter());
    }

    @MessageMapping("/search/unsubscribe/{sourceStr}")
    public void unsubscribe(@DestinationVariable String sourceStr, SubscribeInfo info) {
        String topic = SourceConfig.source2KafkaTopic(SourceConfig.string2Source(sourceStr));
        Optional.ofNullable(scheduleMap.get(topic).remove(info)).ifPresent(f -> f.cancel(true));
        queueMap.get(topic).remove(info);
        log.info("用户 {} 取消订阅", info.getUid());
    }

    public void parseKafkaMessageSubscribe(TmResult telemetryDataProcessResult, String topic) {
        if (queueMap.get(topic).isEmpty()) return;
        ArrayList<EpduResult> epduResults = telemetryDataProcessResult.getEpduResults();
        String station = telemetryDataProcessResult.getStation();

        for (Map.Entry<SubscribeInfo, BlockingQueue<EpduResultForCK>> entry : queueMap.get(topic).entrySet()) {
            SubscribeInfo info = entry.getKey();
            if (!info.isChart()) continue;

            List<ParameterResult> res = epduResults.stream()
                    .flatMap(e -> e.getParameterResults().stream())
                    .filter(p -> p != null && info.getParameter().contains(p.getID()))
                    .toList();

            if (!res.isEmpty()) {
                EpduResultForCK result = new EpduResultForCK();
                result.setParameterResults(new ArrayList<>(res));
                result.setReceiveTime(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS").format(new Date()));
                result.setSatelliteTime(epduResults.get(0).getSatelliteTime());
                result.setStation(station);

                try {
                    entry.getValue().put(result); // 阻塞式插入
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }

    private void sendChartData(String topic, String sourceStr, SubscribeInfo info) {
        BlockingQueue<EpduResultForCK> queue = queueMap.get(topic).get(info);
        if (queue == null || queue.isEmpty()) return;

        List<EpduResultForCK> batch = new ArrayList<>();
        queue.drainTo(batch);
        if (batch.isEmpty()) return;

        try {
            String station = batch.get(0).getStation();
            String destination = "/queue" + station + "/" + sourceStr + "/" + info.getType() + info.getUuid();
            messagingTemplate.convertAndSendToUser(info.getUid(), destination, objectMapper.writeValueAsString(batch));
            log.info("向用户 {} 推送 chart 数据：{}", info.getUid(), batch.size());
        } catch (Exception e) {
            log.error("发送 chart 数据失败", e);
        }
    }
}
```

------

## ✅ 你得到了什么？

| 原设计问题                | 重构后改进方式                              |
| ------------------------- | ------------------------------------------- |
| `List + Lock + Condition` | → `BlockingQueue` 自动线程安全              |
| 每次都要 signal 唤醒线程  | → 不需要，`scheduleWithFixedDelay` 定时发送 |
| 加锁、清空、判断复杂      | → `queue.drainTo()` 一行搞定                |
| 可维护性差、线程泄露风险  | → 使用 `ScheduledFuture.cancel()` 控制线程  |

------

## ✅ 你接下来可以做的

- 将 `parseKafkaMessageSubscribe` 在 Kafka 监听器中调用即可实现实时接收推送；
- 如需同时支持 `list` 模式，可以单独维护一个 `Map<SubscribeInfo, Map<Integer, EpduResultForCK>>`，并走另一套逻辑；
- 如需自定义推送频率，支持用户自定义 `cacheTime`。

------

需要我继续帮你把 Kafka 消费监听器也写好吗？或者你想配合 Apache Camel + Kafka 使用？