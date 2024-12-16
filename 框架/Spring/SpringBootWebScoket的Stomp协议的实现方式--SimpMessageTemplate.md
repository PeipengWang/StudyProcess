# 实现流程实例--包括前端和后端

1、引入依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.51</version>
            <scope>compile</scope>
        </dependency>

    </dependencies>
```

2、websocket配置

添加点对点消息的支持

点对点消息通常使用 `/user/{destination}` 作为目标地址。例如，用户 A 给用户 B 发送一条私人消息。

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
@EnableWebSocketMessageBroker  //启用基于消息代理（Message Broker）的 WebSocket 支持。
@CrossOrigin
@Slf4j
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

//使用一个线程安全的 Set (Collections.synchronizedSet) 来保存已连接的用户 ID。
//                                                    当用户连接或断开时，将其添加或移除到 userIdSet。
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

# 详解

## 接口配置类WebSocketMessageBrokerConfigurer

    public interface WebSocketMessageBrokerConfigurer {
    // 添加这个Endpoint，这样在网页中就可以通过websocket连接上服务,也就是我们配置websocket的服务地址,并且可以指定是否使用socketjs
    void registerStompEndpoints(StompEndpointRegistry var1);
    
    // 配置发送与接收的消息参数，可以指定消息字节大小，缓存大小，发送超时时间
    void configureWebSocketTransport(WebSocketTransportRegistration var1);
    
    // 设置输入消息通道的线程数，默认线程为1，可以自己自定义线程数，最大线程数，线程存活时间
    void configureClientInboundChannel(ChannelRegistration var1);
    
    // 设置输出消息通道的线程数，默认线程为1，可以自己自定义线程数，最大线程数，线程存活时间
    void configureClientOutboundChannel(ChannelRegistration var1);
    
    // 添加自定义的消息转换器，spring 提供多种默认的消息转换器，返回false,不会添加消息转换器，返回true，会添加默认的消息转换器，当然也可以把自己写的消息转换器添加到转换链中
    boolean configureMessageConverters(List<MessageConverter> var1);
    
    // 配置消息代理，哪种路径的消息会进行代理处理
    void configureMessageBroker(MessageBrokerRegistry var1);
    
    // 自定义控制器方法的参数类型，有兴趣可以百度google HandlerMethodArgumentResolver这个的用法
    void addArgumentResolvers(List<HandlerMethodArgumentResolver> var1);
    
    // 自定义控制器方法返回值类型，有兴趣可以百度google HandlerMethodReturnValueHandler这个的用法
    void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> var1);
    }
完整配置类

```
import com.b505.interceptor.WsHandshakeInterceptor;
import com.b505.interceptor.WsChannelInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.converter.MessageConverter;
import org.springframework.messaging.handler.invocation.HandlerMethodArgumentResolver;
import org.springframework.messaging.handler.invocation.HandlerMethodReturnValueHandler;
import org.springframework.messaging.simp.config.ChannelRegistration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketTransportRegistration;

import java.util.List;
/**
 * <配置基于STOMP的websocket>
 **/
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketStompConfig implements WebSocketMessageBrokerConfigurer {

    /**
     * 添加这个Endpoint，这样在网页中就可以通过websocket连接上服务,
     * 也就是我们配置websocket的服务地址,并且可以指定是否使用socketjs
     * @param registry
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry)
    {
        
        /*
         * 1. 将 /serviceName/stomp/websocketJs路径注册为STOMP的端点，
         *    用户连接了这个端点后就可以进行websocket通讯，支持socketJs
         * 2. setAllowedOrigins("*")表示可以跨域
         * 3. withSockJS()表示支持socktJS访问
         * 4. addInterceptors 添加自定义拦截器，这个拦截器是上一个demo自己定义的获取httpsession的拦截器
         */
        registry.addEndpoint("/stomp/websocketJS")
                .setAllowedOrigins("*")
                .addInterceptors(new WebSocketHandshakeInterceptor())
                .withSockJS()

        ;

        /*
         * 看了下源码，它的实现类是WebMvcStompEndpointRegistry ，
         * addEndpoint是添加到WebMvcStompWebSocketEndpointRegistration的集合中，
         * 所以可以添加多个端点
         */
        registry.addEndpoint("/stomp/websocket");
    }

    /**
     * 配置消息代理
     * @param registry
     */
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry)
    {
        /*
         *  enableStompBrokerRelay 配置外部的STOMP服务，需要安装额外的支持 比如rabbitmq或activemq
         * 1. 配置代理域，可以配置多个，此段代码配置代理目的地的前缀为 /topicTest 或者 /userTest
         *    我们就可以在配置的域上向客户端推送消息
         * 3. 可以通过 setRelayHost 配置代理监听的host,默认为localhost
         * 4. 可以通过 setRelayPort 配置代理监听的端口，默认为61613
         * 5. 可以通过 setClientLogin 和 setClientPasscode 配置账号和密码
         * 6. setxxx这种设置方法是可选的，根据业务需要自行配置，也可以使用默认配置
         */
        //registry.enableStompBrokerRelay("/topicTest","/userTest")
                //.setRelayHost("rabbit.someotherserver")
                //.setRelayPort(62623);
                //.setClientLogin("userName")
                //.setClientPasscode("password")
                //;

        // 自定义调度器，用于控制心跳线程
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        // 线程池线程数，心跳连接开线程
        taskScheduler.setPoolSize(1);
        // 线程名前缀
        taskScheduler.setThreadNamePrefix("websocket-heartbeat-thread-");
        // 初始化
        taskScheduler.initialize();

        /*
         * spring 内置broker对象
         * 1. 配置代理域，可以配置多个，此段代码配置代理目的地的前缀为 /topicTest 或者 /userTest
         *    我们就可以在配置的域上向客户端推送消息
         * 2，进行心跳设置，第一值表示server最小能保证发的心跳间隔毫秒数, 第二个值代码server希望client发的心跳间隔毫秒数
         * 3. 可以配置心跳线程调度器 setHeartbeatValue这个不能单独设置，不然不起作用，要配合setTaskScheduler才可以生效
         *    调度器我们可以自己写一个，也可以自己使用默认的调度器 new DefaultManagedTaskScheduler()
         */
        registry.enableSimpleBroker("/topicTest","/userTest")
                .setHeartbeatValue(new long[]{10000,10000})
                .setTaskScheduler(taskScheduler);
        /*
         *  "/app" 为配置应用服务器的地址前缀，表示所有以/app 开头的客户端消息或请求
         *  都会路由到带有@MessageMapping 注解的方法中
         */
        registry.setApplicationDestinationPrefixes("/app");

        /*
         *  1. 配置一对一消息前缀， 客户端接收一对一消息需要配置的前缀 如“'/user/'+userid + '/message'”，
         *     是客户端订阅一对一消息的地址 stompClient.subscribe js方法调用的地址
         *  2. 使用@SendToUser发送私信的规则不是这个参数设定，在框架内部是用UserDestinationMessageHandler处理，
         *     而不是而不是 AnnotationMethodMessageHandler 或  SimpleBrokerMessageHandler
         *     or StompBrokerRelayMessageHandler，是在@SendToUser的URL前加“user+sessionId"组成
         */
        registry.setUserDestinationPrefix("/user");

        /*
         * 自定义路径分割符
         * 注释掉的这段代码添加的分割符为. 分割是类级别的@messageMapping和方法级别的@messageMapping的路径
         * 例如类注解路径为 “topic”,方法注解路径为“hello”，那么客户端JS stompClient.send 方法调用的路径为“/app/topic.hello”
         * 注释掉此段代码后，类注解路径“/topic”,方法注解路径“/hello”,JS调用的路径为“/app/topic/hello”
         */
        //registry.setPathMatcher(new AntPathMatcher("."));

    }

    /**
     * 配置发送与接收的消息参数，可以指定消息字节大小，缓存大小，发送超时时间
     * @param registration
     */
    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        /*
         * 1. setMessageSizeLimit 设置消息缓存的字节数大小 字节
         * 2. setSendBufferSizeLimit 设置websocket会话时，缓存的大小 字节
         * 3. setSendTimeLimit 设置消息发送会话超时时间，毫秒
         */
        registration.setMessageSizeLimit(10240)
                    .setSendBufferSizeLimit(10240)
                    .setSendTimeLimit(10000);
    }

    /**
     * 设置输入消息通道的线程数，默认线程为1，可以自己自定义线程数，最大线程数，线程存活时间
     * @param registration
     */
    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {

        /*
         * 配置消息线程池
         * 1. corePoolSize 配置核心线程池，当线程数小于此配置时，不管线程中有无空闲的线程，都会产生新线程处理任务
         * 2. maxPoolSize 配置线程池最大数，当线程池数等于此配置时，不会产生新线程
         * 3. keepAliveSeconds 线程池维护线程所允许的空闲时间，单位秒
         */
        registration.taskExecutor().corePoolSize(10)
                    .maxPoolSize(20)
                    .keepAliveSeconds(60);
        /*
         * 添加stomp自定义拦截器，可以根据业务做一些处理
         * springframework 4.3.12 之后版本此方法废弃，代替方法 interceptors(ChannelInterceptor... interceptors)
         * 消息拦截器，实现ChannelInterceptor接口
         */
        registration.setInterceptors(webSocketChannelInterceptor());
    }

    /**
     *设置输出消息通道的线程数，默认线程为1，可以自己自定义线程数，最大线程数，线程存活时间
     * @param registration
     */
    @Override
    public void configureClientOutboundChannel(ChannelRegistration registration) {
        registration.taskExecutor().corePoolSize(10)
                    .maxPoolSize(20)
                    .keepAliveSeconds(60);
        //registration.setInterceptors(new WebSocketChannelInterceptor());
    }

    /**
     * 添加自定义的消息转换器，spring 提供多种默认的消息转换器，
     * 返回false,不会添加消息转换器，返回true，会添加默认的消息转换器，当然也可以把自己写的消息转换器添加到转换链中
     * @param list
     * @return
     */
    @Override
    public boolean configureMessageConverters(List<MessageConverter> list) {
        return true;
    }

    /**
     * 自定义控制器方法的参数类型，有兴趣可以百度google HandlerMethodArgumentResolver这个的用法
     * @param list
     */
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> list) {

    }

    /**
     * 自定义控制器方法返回值类型，有兴趣可以百度google HandlerMethodReturnValueHandler这个的用法
     * @param list
     */
    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> list) {

    }

    /**
     * 拦截器加入 spring ioc容器
     * @return
     */
    @Bean
    public WebSocketChannelInterceptor webSocketChannelInterceptor()
    {
        return new WebSocketChannelInterceptor();
    }
}


```

## 新建保存用户认证信息的实体类MyPrincipalHandshakeHandler

上面的代码可以看到，我们在STOMP端点上注册了一个保存用户认证信息的实体类MyPrincipalHandshakeHandler和一个握手拦截器WebSocketHandshakeInterceptor

用户认证信息类MyPrincipalHandshakeHandler需要实现Principal接口

```
import java.security.Principal;

/**
 * <websocket登录连接对象>
 * <用于保存websocket连接过程中需要存储的业务参数>
 * @author 郑智聪
 * @version 2018-06-11
 **/
public class MyPrincipalHandshakeHandler implements Principal {

    /**
     * 用户身份标识符
     */
    private String token;

    public WebSocketUserAuthentication(String token) {
        this.token = token;
    }

    /**
     * 获取用户登录令牌
     * @return
     */
    @Override
    public String getName() {
        return token;
    }
}


```

## 握手拦截器WebSocketHandshakeInterceptor

握手拦截器WebSocketHandshakeInterceptor需要继承HttpSessionHandshakeInterceptor，它是HandshakeInterceptor接口的一个实现，并重写beforeHandshake方法，实现自己的握手拦截的逻辑。

```
import org.apache.commons.lang.StringUtils;
import org.apache.log4j.Logger;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.support.DefaultHandshakeHandler;

import javax.servlet.http.HttpSession;
import java.security.Principal;
import java.util.Map;

/**
 * <设置认证用户信息的握手拦截器>
 **/
public class MyPrincipalHandshakeHandler extends HttpSessionHandshakeInterceptor{

    private static final Logger log = Logger.getLogger(MyPrincipalHandshakeHandler.class);

     @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) {
	  //这里就是简单 
      HttpSession session = getSession(request);
		if (session != null) {
			if (isCopyHttpSessionId()) {
				attributes.put(HTTP_SESSION_ID_ATTR_NAME, session.getId());
			}
			Enumeration<String> names = session.getAttributeNames();
			while (names.hasMoreElements()) {
				String name = names.nextElement();
				if (isCopyAllAttributes() || getAttributeNames().contains(name)) {
					attributes.put(name, session.getAttribute(name));
				}
			}
		}
		return true;
    }

    private HttpSession getSession(ServerHttpRequest request) {
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest serverRequest = (ServletServerHttpRequest) request;
            return serverRequest.getServletRequest().getSession(false);
        }
        return null;
    }
}


```

## 消息通道拦截器

除此之外，如果需要添加监听，我们的监听类需要实现ChannelInterceptor接口，在 springframework包5.0.7之前这一步我们一般是实现ChannelInterceptorAdapter 抽象类，不过这个类已经废弃了，文档也推荐直接实现接口。

在ChannelInterceptor接口中的preSend能在消息发送前做一些处理，例如可以获取到用户登录的唯一token令牌，这里的令牌是我们业务传递给客户端的，例如用户在登录成功后我们后台生成的一个标识符，客户端在和服务端建立websocket连接的时候，我们可以从消息头中获取到这种业务参数，并做一系列后续处理。

```
import org.apache.log4j.Logger;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.simp.stomp.StompCommand;
import org.springframework.messaging.simp.stomp.StompHeaderAccessor;
import org.springframework.messaging.support.ChannelInterceptor;
import org.springframework.messaging.support.MessageHeaderAccessor;

import javax.servlet.http.HttpSession;

import static org.springframework.messaging.simp.stomp.StompCommand.CONNECT;

/**
 * <websocke消息监听，用于监听websocket用户连接情况>
 * <功能详细描述>
 **/
public class WebSocketChannelInterceptor implements ChannelInterceptor {


    Logger log = Logger.getLogger(WebSocketChannelInterceptor.class);

    // 在消息发送之前调用，方法中可以对消息进行修改，如果此方法返回值为空，则不会发生实际的消息发送调用
    @Override
    public Message<?> preSend(Message<?> message, MessageChannel messageChannel) {

        StompHeaderAccessor accessor =  MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
        /**
         * 1. 判断是否为首次连接请求，如果已经连接过，直接返回message
         * 2. 网上有种写法是在这里封装认证用户的信息，本文是在http阶段，websockt 之前就做了认证的封装，所以这里直接取的信息
         */
        if(StompCommand.CONNECT.equals(accessor.getCommand()))
        {
            /*
             * 1. 这里获取就是JS stompClient.connect(headers, function (frame){.......}) 中header的信息
             * 2. JS中header可以封装多个参数，格式是{key1:value1,key2:value2}
             * 3. header参数的key可以一样，取出来就是list
             * 4. 样例代码header中只有一个token，所以直接取0位
             */
            String token = accessor.getNativeHeader("token").get(0);

            /*
             * 1. 这里直接封装到StompHeaderAccessor 中，可以根据自身业务进行改变
             * 2. 封装大搜StompHeaderAccessor中后，可以在@Controller / @MessageMapping注解的方法中直接带上StompHeaderAccessor
             *    就可以通过方法提供的 getUser()方法获取到这里封装user对象
             * 2. 例如可以在这里拿到前端的信息进行登录鉴权
             */
            WebSocketUserAuthentication user = (WebSocketUserAuthentication) accessor.getUser();

            System.out.println("认证用户:" + user.toString() + " 页面传递令牌" + token);

        }else if (StompCommand.DISCONNECT.equals(accessor.getCommand()))
        {
			System.out.println("用户:" + accessor.getUser() + " 断开连接");
        }
        return message;
    }

    // 在消息发送后立刻调用，boolean值参数表示该调用的返回值
    @Override
    public void postSend(Message<?> message, MessageChannel messageChannel, boolean b) {

        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);

        /*
         * 拿到消息头对象后，我们可以做一系列业务操作
         * 1. 通过getSessionAttributes()方法获取到websocketSession，
         *    就可以取到我们在WebSocketHandshakeInterceptor拦截器中存在session中的信息
         * 2. 我们也可以获取到当前连接的状态，做一些统计，例如统计在线人数，或者缓存在线人数对应的令牌，方便后续业务调用
         */
        HttpSession httpSession = (HttpSession) accessor.getSessionAttributes().get("HTTP_SESSION");

        // 这里只是单纯的打印，可以根据项目的实际情况做业务处理
        log.info("postSend 中获取httpSession key：" + httpSession.getId());

        // 忽略心跳消息等非STOMP消息
        if(accessor.getCommand() == null)
        {
            return;
        }

        // 根据连接状态做处理，这里也只是打印了下，可以根据实际场景，对上线，下线，首次成功连接做处理
        System.out.println(accessor.getCommand());
        switch (accessor.getCommand())
        {
            // 首次连接
            case CONNECT:
                log.info("httpSession key：" + httpSession.getId() + " 首次连接");
                break;
            // 连接中
            case CONNECTED:
                break;
            // 下线
            case DISCONNECT:
                log.info("httpSession key：" + httpSession.getId() + " 下线");
                break;
             default:
                break;
        }


    }
    /*
     * 1. 在消息发送完成后调用，而不管消息发送是否产生异常，在次方法中，我们可以做一些资源释放清理的工作
     * 2. 此方法的触发必须是preSend方法执行成功，且返回值不为null,发生了实际的消息推送，才会触发
     */
    @Override
    public void afterSendCompletion(Message<?> message, MessageChannel messageChannel, boolean b, Exception e) {

    }

    /* 1. 在消息被实际检索之前调用，如果返回false,则不会对检索任何消息，只适用于(PollableChannels)，
     * 2. 在websocket的场景中用不到
     */
    @Override
    public boolean preReceive(MessageChannel messageChannel) {
        return true;
    }

    /*
     * 1. 在检索到消息之后，返回调用方之前调用，可以进行信息修改，如果返回null,就不会进行下一步操作
     * 2. 适用于PollableChannels，轮询场景
     */
    @Override
    public Message<?> postReceive(Message<?> message, MessageChannel messageChannel) {
        return message;
    }

    /*
     * 1. 在消息接收完成之后调用，不管发生什么异常，可以用于消息发送后的资源清理
     * 2. 只有当preReceive 执行成功，并返回true才会调用此方法
     * 2. 适用于PollableChannels，轮询场景
     */
    @Override
    public void afterReceiveCompletion(Message<?> message, MessageChannel messageChannel, Exception e) {
    }
}


```

## 服务端消息处理器

1. MessageMapping
   Spring对于WebSocket封装的特别简单，提供了一个@MessageMapping注解，功能类似@RequestMapping，它是存在于Controller中的，定义一个消息的基本请求，功能也跟@RequestMapping类似，包括支持通配符``的url定义等等，详细用法参见Annotation Message Handling

```
@MessageMapping("/change-notice")
@SendTo("/topic/notice")
public String greeting(String value) {    
    return value;
}

```

1. SimpMessagingTemplate
   SimpMessagingTemplate是Spring-WebSocket内置的一个消息发送工具，可以将消息发送到指定的客户端。

```
@Autowired
private SimpMessagingTemplate messagingTemplate;

```

@SendTo定义了消息的目的地。结合例子解释就是“接收/app/change-notice发来的value，然后将value转发到/topic/notice客户端。

/topic/notice是客户端发起连接后，订阅服务端消息时指定的一个地址，用于接收服务端的返回。

### 点对点发送

```
/**
     * 根据ID 把消息推送给指定用户
     * 1. 这里用了 @SendToUser 和 返回值 其意义是可以在发送成功后回执给发送放其信息发送成功
     * 2. 非必须，如果实际业务不需要关心此，可以不用@SendToUser注解，方法返回值为void
     * 3. 这里接收人的参数是用restful风格带过来了，websocket把参数带到后台的方式很多，除了url路径，
     *    还可以在header中封装用@Header或者@Headers去取等多种方式
     * @param accountId 消息接收人ID
     * @param json 消息JSON字符串
     * @param headerAccessor
     * @return
     */
    @MessageMapping("/sendChatMsgById/{accountId}")
    @SendToUser(value = "/userTest/callBack")
    public Map<String, Object> sendChatMsgById(
        @DestinationVariable(value = "accountId") String accountId, String json,
        StompHeaderAccessor headerAccessor)
    {
        Map msg = (Map)JSON.parse(json);
        Map<String, Object> data = new HashMap<String, Object>();

        // 这里拿到的user对象是在WebSocketChannelInterceptor拦截器中绑定上的对象
        WebSocketUserAuthentication user = (WebSocketUserAuthentication)headerAccessor.getUser();

        log.info("SendToUser controller 中获取用户登录令牌：" + user.getName()
                + " socketId:" + headerAccessor.getSessionId());

        // 向用户发送消息,第一个参数是接收人、第二个参数是浏览器订阅的地址，第三个是消息本身
        // 如果服务端要将消息发送给特定的某一个用户，
        // 可以使用SimpMessageTemplate的convertAndSendToUser方法(第一个参数是用户的登陆名username)
        String address = "/userTest/callBack";
        messagingTemplate.convertAndSendToUser(accountId, address, msg.get("msg"));

        data.put("msg", "callBack 消息已推送，消息内容：" + msg.get("msg"));
        return data;
    }


```

# 前端VUE实现

vue项目需要先引入sockjs-client和stompjs这两个库

```
// 安装并引入相关模块
import SockJS from 'sockjs-client';
import Stomp from 'stompjs';
import {getToken} from './auth'//这是我自己的获取token的方法，可以根据具体业务进行修改

export default {
  methods: {
    initWebSocket() {
      this.connection();
      let self = this;
      // 断开重连机制,尝试发送消息,捕获异常发生时重连
      this.timer = setInterval(() => {
        try {
          self.stompClient.send("alive");
        } catch (err) {
          console.log("断线了: " + err);
          self.connection();
        }
      }, 5000);
    },
    connection() {
      // 建立连接对象
      this.socket = new SockJS('http://127.0.0.1:8081/websocket');//连接服务端提供的通信接口，连接以后才可以订阅广播消息和个人消息，注意这里用的是http而不是原生WebSocket的ws
      
      // 获取STOMP子协议的客户端对象
      this.stompClient = Stomp.over(this.socket);

      // 定义客户端的认证信息,按需求配置
      const token = getToken();
      let headers = {
        'Authorization': token,
      };
      // 向服务器发起websocket连接
      this.stompClient.connect(headers, this.onConnected, this.onFailed);
    },
    onConnected: function (frame) {
      console.log("Connected: " + frame);
      let topic = "/ws/topic/charger.messageTopic";
      this.stompClient.subscribe(topic,this.onFailed);
    },
    onFailed(frame) {
      console.log("Failed: " + frame);
    },
    disconnect() {
      if (this.stompClient != null) {
        this.stompClient.disconnect();
        console.log("Disconnected");
      }
    }
  }
};


```

