# 代码配置

```
package com.springbootwebsocket.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.config.ChannelRegistration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.ChannelInterceptor;
import org.springframework.messaging.support.MessageHeaderAccessor;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.socket.config.annotation.*;

import java.security.Principal;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;

@Configuration
@EnableWebSocket
@EnableWebSocketMessageBroker
@CrossOrigin
@Slf4j
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    Set<String> userIdSet = Collections.synchronizedSet(new HashSet<>());

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.taskExecutor().corePoolSize(50)
                .maxPoolSize(100)
                .keepAliveSeconds(60);
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    log.info("收到连接请求");
                    if (accessor.containsNativeHeader("uid")) {
                        String uid = accessor.getNativeHeader("uid").get(0);
                        userIdSet.add(uid);
                        log.info("{}已连接", uid);
                        Principal principal = () -> uid;
                        accessor.setUser(principal);
                    }
                } else if (StompCommand.DISCONNECT.equals(accessor.getCommand())) {
                    if (accessor.getUser() != null) userIdSet.remove(accessor.getUser().getName());
                    log.info("用户 {} 已断开连接", accessor.getUser().getName());
                }
                return message;
            }
        });
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws/stomp/tm")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/queue", "/topic");
        registry.setApplicationDestinationPrefixes("/tm");
        registry.setUserDestinationPrefix("/user");
    }

    @Bean
    public Set<String> userIdSet() {
        return userIdSet;
    }
}



```

## 代码解析

### configureClientInboundChannel

这段代码配置了 Spring WebSocket 的 **`ClientInboundChannel`**，主要负责处理 WebSocket 消息的输入通道，并在其中增加了自定义的 **`ChannelInterceptor`** 来拦截消息。在该拦截器中，您可以在连接建立 (`CONNECT`) 和断开连接 (`DISCONNECT`) 时执行一些逻辑。

让我们逐步解析代码：

#### 1. **配置线程池：**

```java
registration.taskExecutor().corePoolSize(50)
                .maxPoolSize(100)
                .keepAliveSeconds(60);
```

这段代码配置了 `ClientInboundChannel` 的 **线程池**。该线程池用于处理 WebSocket 连接请求，处理消息的任务会由这个线程池来执行。

- **`corePoolSize(50)`**: 设置线程池的核心线程数为 50。这意味着线程池会保持至少 50 个线程可用来处理请求。
- **`maxPoolSize(100)`**: 设置线程池的最大线程数为 100。如果核心线程数不足以处理所有任务，线程池将动态增加线程数，直到达到最大线程数 100。
- **`keepAliveSeconds(60)`**: 如果线程池中的线程超过核心线程数并且闲置超过 60 秒，那么这些线程将被回收。

还有其他配置，例如

```
setAllowedOrigins：配置跨域的来源。
setHandshakeHandler：自定义握手处理逻辑。
setInterceptors：设置拦截器（如身份验证拦截）。
setMaxMessageSize、setMessageSizeLimit、setSendTimeLimit：消息传输的限制。
setSubProtocolHandler：配置 WebSocket 子协议的处理器。
setApplicationDestinationPrefixes、setUserDestinationPrefix：设置消息的目标前缀，定义客户端和服务端通信的路径。
```

#### 2. **设置 `ChannelInterceptor`：**

```java
registration.interceptors(new ChannelInterceptor() {
    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
```

`ChannelInterceptor` 是一个消息拦截器，用来处理通过 WebSocket 传输的消息。我们通过实现 `preSend` 方法来拦截消息。

- `StompHeaderAccessor` 提供了访问 STOMP 协议的头部信息，例如连接请求 (`CONNECT`) 和断开请求 (`DISCONNECT`) 时的处理。
- `preSend` 方法会在消息被发送之前被调用。

#### 3. **处理 `CONNECT` 请求：**

```java
if (StompCommand.CONNECT.equals(accessor.getCommand())) {
    log.info("收到连接请求");
    if (accessor.containsNativeHeader("uid")) {
        String uid = accessor.getNativeHeader("uid").get(0);
        userIdSet.add(uid);
        log.info("{}已连接", uid);
        Principal principal = () -> uid;
        accessor.setUser(principal);
    }
}
```

这部分代码是在处理客户端的连接请求时执行的：

- **`StompCommand.CONNECT`**：表示一个连接请求。`accessor.getCommand()` 获取的是 STOMP 请求的命令，`CONNECT` 表示建立 WebSocket 连接。
- **`accessor.containsNativeHeader("uid")`**：检查连接请求的头部中是否包含 `uid`（即用户的唯一标识符）。
- **`String uid = accessor.getNativeHeader("uid").get(0)`**：如果 `uid` 存在，从请求头部中获取该用户的 ID。
- **`userIdSet.add(uid)`**：将该用户的 ID 添加到 `userIdSet` 中，方便后续管理已连接的用户。
- **`Principal principal = () -> uid; accessor.setUser(principal);`**：将 `uid` 设置为当前用户的 `Principal`，这是 WebSocket 连接的用户身份，后续可以通过 `accessor.getUser()` 获取该用户。

通过这种方式，后端可以识别并跟踪每个连接的用户。

#### 4. **处理 `DISCONNECT` 请求：**

```java
} else if (StompCommand.DISCONNECT.equals(accessor.getCommand())) {
    if (accessor.getUser() != null) userIdSet.remove(accessor.getUser().getName());
    log.info("用户 {} 已断开连接", accessor.getUser().getName());
}
```

当客户端断开连接时，会触发 `DISCONNECT` 请求：

- **`StompCommand.DISCONNECT`**：表示客户端断开连接的请求。
- **`accessor.getUser()`**：获取当前用户信息，`getUser()` 会返回之前通过 `setUser` 设置的 `Principal`（此处为 `uid`）。
- **`userIdSet.remove(accessor.getUser().getName())`**：当用户断开连接时，从 `userIdSet` 中移除该用户的 ID。
- **`log.info("用户 {} 已断开连接", accessor.getUser().getName())`**：记录日志，输出该用户已断开连接。

#### 总结

1. **线程池配置**：配置了线程池来处理 WebSocket 的任务，确保能够并发处理多个 WebSocket 连接。
2. **拦截器**：使用 `ChannelInterceptor` 拦截 `CONNECT` 和 `DISCONNECT` 请求。
3. **`CONNECT` 请求处理**：在客户端连接时，获取用户的 `uid`，将其添加到 `userIdSet` 中并设置为当前用户身份。
4. **`DISCONNECT` 请求处理**：在客户端断开时，移除已断开用户的 ID，并记录日志。

#### 注意

1. **资源管理**：为了避免内存泄漏或资源浪费，确保 `userIdSet` 被合理清理，特别是在大量用户连接和断开时。
2. **日志记录**：可以添加更多细粒度的日志记录，便于调试和监控连接的状态。

总体来说，您的配置逻辑非常清晰且高效，实现了 WebSocket 连接的身份认证与管理功能。如果有进一步问题，随时欢迎提问！





```java
@Override
public void configureMessageBroker(MessageBrokerRegistry registry) {
    registry.enableSimpleBroker("/queue", "/topic");
    registry.setApplicationDestinationPrefixes("/tm");
    registry.setUserDestinationPrefix("/user");
}
```

### configureMessageBroker

#### 1. **`enableSimpleBroker("/queue", "/topic")`**

- **启用简单消息代理**：
  - `enableSimpleBroker` 方法启用了一个内存消息代理（**Simple Broker**），这是 Spring 的默认消息代理，适用于小型应用。
  - 这个代理会监听路径 `/queue` 和 `/topic`。这意味着客户端可以订阅这些前缀路径下的主题（topic）和队列（queue）。
  - **`/queue`** 和 **`/topic`** 是消息传递的目标前缀，通常用于区分 **点对点消息**（例如队列 `/queue`）和 **发布/订阅消息**（例如主题 `/topic`）。
  - 订阅 `/topic` 路径的客户端将接收到来自后端的广播消息（即发布/订阅消息模式）。
  - 订阅 `/queue` 路径的客户端将接收到特定于用户的消息（即点对点消息模式）。

#### 2. **`setApplicationDestinationPrefixes("/tm")`**

- **设置应用前缀**：
  - `setApplicationDestinationPrefixes("/tm")` 配置了应用的 **消息目标前缀**。这意味着客户端发送的消息路径应该以 `/tm` 开头。
  - 例如，如果前端发送一个消息到 `/app/tm/message`，这会触发后端的 `@MessageMapping("tm/message")` 处理方法。
  - 所以，**`/tm`** 是应用级别的前缀，用于区分来自不同应用的消息路径。

#### 3. **`setUserDestinationPrefix("/user")`**

- **设置用户消息前缀**：
  - `setUserDestinationPrefix("/user")` 配置了与特定用户相关的消息路径前缀。这个前缀用于为特定用户发送点对点消息。
  - 比如，如果您想给用户发送特定的消息，可以使用 `/user/{userId}/queue/someQueue` 路径。这表示该消息将被发送给指定用户的队列。
  - 客户端可以订阅这个路径，以便接收属于他们自己的消息。例如，用户 1 订阅 `/user/user1/queue/someQueue`，则只有这个用户能接收到该队列的消息。

#### 示例：前后端如何配合

1. **后端处理消息**：
    假设您在后端使用 `@MessageMapping("/tm/message")` 处理消息，客户端会将消息发送到 `/app/tm/message`。
2. **后端发送消息**：
   - **发布/订阅消息**：
      如果您希望广播消息给所有客户端，可以使用 `/topic/` 作为路径。例如：`messagingTemplate.convertAndSend("/topic/response", "消息内容")`，客户端订阅 `/topic/response`，会接收到消息。
   - **点对点消息**：
      如果您希望将消息发送给特定用户，可以使用 `/user/{userId}/queue/` 路径。例如：`messagingTemplate.convertAndSendToUser("user1", "/queue/response", "私人消息")`，只有 `user1` 会收到该消息。

#### 使用场景：

- **`/queue`**：
  - 用于点对点消息，例如给特定用户发送消息。
  - 客户端可以订阅 `/queue/response` 等。
- **`/topic`**：
  - 用于广播消息，例如给所有订阅者推送更新。
  - 客户端可以订阅 `/topic/updates` 等。
- **`/app` 前缀**：
  - 用于接收来自前端的请求，例如 `/app/tm/message`。

#### 总结：

- **`/queue` 和 `/topic`** 用于配置消息的传递类型：点对点消息和发布/订阅消息。
- **`/tm`** 前缀用于接收前端请求消息。
- **`/user`** 前缀用于发送给特定用户的消息。

这种配置方式让您可以通过 WebSocket 在应用中实现强大的实时消息推送功能，包括广播和针对用户的消息推送。

### registerStompEndpoints

这段代码是在 Spring WebSocket 配置中使用的 `registerStompEndpoints` 方法，主要功能是注册 WebSocket 端点，并为该端点配置一些连接选项。这里使用了 **STOMP** 协议和 **SockJS** 作为回退机制。

#### 1. **`registry.addEndpoint("/ws/stomp/tm")`**

- **目的**：配置 WebSocket 连接的端点。
- **功能**：这行代码定义了一个 WebSocket 连接的 **端点 URL**，客户端需要通过这个 URL 来建立 WebSocket 连接。
- **端点路径**：`/ws/stomp/tm`
  - 这表示 WebSocket 连接的 **路径** 是 `/ws/stomp/tm`。前端需要连接到这个 URL 才能建立 WebSocket 连接。
  - 客户端会通过 `SockJS` 或 `WebSocket` 协议连接到此端点。

**示例**：

- 客户端在浏览器中连接时需要使用 `ws://<server-url>/ws/stomp/tm`，其中 `<server-url>` 是后端服务的地址。

#### 2. **`setAllowedOriginPatterns("\*")`**

- **目的**：配置跨域请求的来源。

- **功能**：`setAllowedOriginPatterns("*")` 表示允许来自 **任何来源** 的 WebSocket 连接请求。

- **跨域配置**：默认情况下，浏览器会限制不同源之间的 WebSocket 连接。通过设置 `setAllowedOriginPatterns("*")`，服务器允许所有来源的客户端与其建立连接。

  - **`\*`** 代表所有来源，即没有来源限制，允许来自任何域的客户端连接到 WebSocket 服务。

- **安全性问题**：在实际应用中，为了安全起见，您通常不希望允许所有来源的连接，特别是在生产环境中。可以将 `*` 改为特定的域名，如：

  ```
  java复制编辑registry.addEndpoint("/ws/stomp/tm")
          .setAllowedOriginPatterns("http://example.com")
          .withSockJS();
  ```

  这样只允许来自 `http://example.com` 的客户端连接。

#### 3. **`withSockJS()`**

- **目的**：启用 SockJS 协议作为 WebSocket 连接的回退机制。
- **功能**：SockJS 是一个 JavaScript 库，提供了 WebSocket 的替代方案，以便在某些浏览器或网络环境中，WebSocket 不能正常使用时，使用其他技术来模拟 WebSocket 功能（例如，通过长轮询、AJAX 请求等）。
- **回退机制**：如果客户端不支持 WebSocket，SockJS 会尝试使用其他方式进行连接，确保连接不会因为 WebSocket 不支持而失败。
- **`withSockJS()`** 表示在 WebSocket 连接失败时，会回退到 SockJS 提供的其他协议（例如，XHR 请求、AJAX 长轮询等）。