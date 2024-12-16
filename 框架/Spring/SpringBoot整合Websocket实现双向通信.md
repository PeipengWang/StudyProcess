# SpringBoot整合Websocket实现双向通信

## 需求描述：

1、前台能够建立起与后台的websocket

2、后台向前台不断推送数据，并且在前台展示

3、前台向后台发送数据点击按键，后台打印日志

## 后台实现

### 建立项目 导入依赖

```xml
 <properties>
        <java.version>8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.51</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>

```

### 启动

```java
package com.example.demo_websocket;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication

public class DemoWebsocketApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoWebsocketApplication.class, args);
    }
}

```

### 配置类引入websocket

```java
package com.example.demo_websocket.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
@EnableWebSocket
public class WebSocketConfiguration {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

### 建立websocket服务器处理端

```java
package com.example.demo_websocket.server;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

@ServerEndpoint("/websocket/{userId}")
@Component
public class WebSocketServer {

//    private static final Logger log = (Logger) LoggerFactory.getLogger(WebSocketServer.class);

    /**
     * 当前在线连接数
     */
    private static AtomicInteger onlineCount = new AtomicInteger(0);

    /**
     * 用来存放每个客户端对应的 WebSocketServer 对象
     */
    private static ConcurrentHashMap<String, WebSocketServer> webSocketMap = new ConcurrentHashMap<>();

    /**
     * 与某个客户端的连接会话，需要通过它来给客户端发送数据
     */
    private Session session;

    /**
     * 接收 userId
     */
    private String userId = "";

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session, @PathParam("userId") String userId) {
        this.session = session;
        this.userId = userId;
        if (webSocketMap.containsKey(userId)) {
            webSocketMap.remove(userId);
            webSocketMap.put(userId, this);
        } else {
            webSocketMap.put(userId, this);
            addOnlineCount();
        }
        System.out.println("用户连接:" + userId + ",当前在线人数为:" + getOnlineCount());
        try {
            sendMessage("连接成功！");
            int i = 0;
            while (i < 10){
                Thread.sleep(1000);
                sendMessage("推送消息：" + i);
                i++;
            }
        } catch (IOException | InterruptedException e) {
            System.out.println("用户:" + userId + ",网络异常!!!!!!");
        }
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        if (webSocketMap.containsKey(userId)) {
            webSocketMap.remove(userId);
            subOnlineCount();
        }
        System.out.println("用户退出:" + userId + ",当前在线人数为:" + getOnlineCount());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("用户消息:" + userId + ",报文:" + message);
    }

    /**
     * 发生错误时调用
     *
     * @param session
     * @param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        System.out.println("用户错误:" + this.userId + ",原因:" + error.getMessage());
        error.printStackTrace();
    }

    /**
     * 实现服务器主动推送
     */
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }

    public static synchronized AtomicInteger getOnlineCount() {
        return onlineCount;
    }

    public static synchronized void addOnlineCount() {
        WebSocketServer.onlineCount.getAndIncrement();
    }

    public static synchronized void subOnlineCount() {
        WebSocketServer.onlineCount.getAndDecrement();
    }
}

```

## 前台

接下来启动项目，使用 WebSocket 在线测试工具（`http://www.easyswoole.com/wstool.html`）进行测试。

打开网页后，在服务地址中输入`ws://127.0.0.1:8080/websocket/wupx`，点击`开启连接`按钮，消息记录中会多一条由服务器端发送的`连接成功！`记录。

接下来再打开一个网页，服务地址中输入`ws://127.0.0.1:8080/websocket/huxy`，点击`开启连接`按钮，然后回到第一次打开的网页在消息框中输入`{"toUserId":"huxy","message":"i love you"}`，点击`发送到服务端`，第二个网页中会收到服务端推送的消息`{"fromUserId":"wupx","message":"i love you","toUserId":"huxy"}`。



直接写一个html界面

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>WebSocket Demo</title>
</head>
<body>
<h1>WebSocket Demo</h1>
<input type="text" id="message" placeholder="Enter message">
<button onclick="sendMessage()">Send</button>
<div id="response"></div>

<script>
  let ws;

  function connect() {
    ws = new WebSocket('ws://127.0.0.1:8080/websocket/wpp');

    ws.onopen = function() {
      console.log('Connected to WebSocket server');
    };

    ws.onmessage = function(event) {
      console.log('Received:', event.data);
      document.getElementById('response').innerText = event.data;
    };

    ws.onclose = function() {
      console.log('Disconnected from WebSocket server');
    };
  }

  function sendMessage() {
    const message = document.getElementById('message').value;
    ws.send(message);
  }

  window.onload = connect;
</script>
</body>
</html>

```

这样就完成了一个websocket的双向测试

## 总结

websocket无论在前台还是后台都会通过事件调用的方式来执行逻辑代码：

- @OnOpen：标注客户端打开 WebSocket 服务端点调用方法
- @OnClose：标注客户端关闭 WebSocket 服务端点调用方法
- @OnMessage：标注客户端发送消息，WebSocket 服务端点调用方法
- @OnError：标注客户端请求 WebSocket 服务端点发生异常调用方法

分别对应着html的ws.onopen 、ws.onclose、 ws.onmessage、ws、onerror

当然发送后台是this.session.getBasicRemote().sendText(message);对应着前台的ws.send(message);