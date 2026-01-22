是的，**很可能需要修改配置** —— 但不是简单地“改地址”，而是要 **确保 RocketMQ Broker 向 NameServer 注册的是 Dashboard 能访问的 IP 和正确的端口（通常是 `10911`，而不是 `10909`）**。

你提到 `10.2.0.10:10909` 是你的**内网地址**，而 Dashboard 运行在另一台机器（IP `26.26.26.1`），这说明存在 **网络可达性** 或 **Broker 注册地址不正确** 的问题。

------

### 🔍 核心问题分析

RocketMQ 的工作流程如下：

1. **Broker 启动时**，会向 NameServer 注册自己的 **IP + 端口**。
2. **Dashboard（或任何客户端）** 先连 NameServer 获取 Broker 地址列表。
3. **然后直接连接该 Broker 地址** 执行管理命令（如查 Topic、统计等）。

> ❗ 如果 Broker 注册的是 **内网 IP（如 10.2.0.10）**，而 Dashboard 在另一个网络（如公网、不同 VPC、Docker 容器等），**且无法路由到该内网 IP**，就会出现：
>
> - `RemotingSendRequestException`
> - `TimeoutException`

此外，**端口 `10909` 很可疑** —— 它通常不是客户端通信端口。

------

### ✅ 正确做法：让 Broker 注册 **Dashboard 可访问的 IP + 正确端口**

#### 第一步：确认 Broker 的 `listenPort`

在 `broker.conf` 中，确保：

```properties
# 客户端（Producer/Consumer/Dashboard）连接的端口，必须是这个！
listenPort=10911
```

> ⚠️ `10909` 是 RocketMQ 5.x Controller 或 DLedger 内部端口，**Dashboard 不应连接它**。

------

#### 第二步：强制 Broker 注册一个 Dashboard 能访问的 IP

在 `broker.conf` 中显式指定对外 IP：

```properties
# 关键配置！告诉 Broker：“你对外的地址是这个”
brokerIP1=154.8.237.182   # ← 替换为 Broker 服务器的公网 IP 或 Dashboard 能访问的 IP

# 如果有多个网卡，还可以指定 brokerIP2（用于 HA）
# brokerIP2=...

# 确保端口正确
listenPort=10911
```

> 💡 `brokerIP1` 的作用：**覆盖自动探测的内网 IP（如 10.2.0.10）**，让 NameServer 存储的是你指定的 IP。

------

#### 第三步：重启 Broker

```bash
# 先停
pkill -f BrokerStartup

# 清理锁（可选）
rm -f ~/store/lock

# 启动（记得加载 conf）
nohup bin/mqbroker -n 154.8.237.182:9876 -c conf/broker.conf &
```

------

#### 第四步：验证注册地址是否正确

在任意机器上执行：

```bash
bin/mqadmin clusterList -n 154.8.237.182:9876
```

✅ 正确输出应类似：

```
#Cluster Name     #Broker Name    #Addr
DefaultCluster    broker-a        154.8.237.182:10911   ← 注意是 10911，且 IP 是你配置的
```

❌ 错误输出（当前问题）：

```
DefaultCluster    broker-a        10.2.0.10:10909
```

------

#### 第五步：从 Dashboard 机器测试连通性

```bash
telnet 154.8.237.182 10911
```

如果通了，Dashboard 就能正常工作。

------

### 🌐 网络架构建议

| 场景                                             | 解决方案                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ |
| **Dashboard 和 Broker 在同一内网**               | 确保 `brokerIP1` 设为内网 IP（如 `10.2.0.10`），且防火墙开放 `10911` |
| **Dashboard 在公网，Broker 在内网（有公网 IP）** | `brokerIP1=公网IP`，安全组开放 `10911`                       |
| **Broker 在 NAT/容器/云主机**                    | 必须通过 `brokerIP1` 指定外部可访问地址                      |
| **使用 Docker**                                  | 需用 `-e brokerIP1=宿主机IP`，并映射 `10911` 端口            |

------

### ❌ 不要这样做

- 不要试图在 Dashboard 里“硬编码”Broker 地址（它从 NameServer 动态获取）。
- 不要连接 `10909` 端口（除非你明确知道它是 Remoting 端口 —— 通常不是）。

------

### 📝 总结：你需要做的

1. 编辑 `broker.conf`

   ：

   ```properties
   brokerIP1=154.8.237.182   # ← Dashboard 能访问的 IP
   listenPort=10911          # ← 必须是这个端口
   ```

2. **重启 Broker**

3. **用 `mqadmin clusterList` 验证注册地址**

4. **从 Dashboard 机器 `telnet 154.8.237.182 10911` 测试**

> 完成后，Dashboard 就不会再尝试连接 `10.2.0.10:10909`，而是连接 `154.8.237.182:10911`，问题解决。

如果你不确定 `154.8.237.182` 是否合适（比如它是 NameServer 的 IP？），请告诉我：

- Broker 服务器的公网 IP 或内网 IP（对 Dashboard 可见的）
- 网络拓扑（是否同 VPC？是否跨云？）

我可以帮你写具体的 `broker.conf`。

保存信息

```
[2026-01-20 09:29:01.218] INFO scheduling-1 - create MQAdmin instance ClientConfig [namesrvAddr=154.8.237.182:9876, clientIP=26.26.26.1, instanceName=1768872540384, clientCallbackExecutorThreads=12, pollNameServerInterval=30000, heartbeatBrokerInterval=30000, persistConsumerOffsetInterval=5000, pullTimeDelayMillsWhenException=1000, unitMode=false, unitName=null, vipChannelEnabled=true, useTLS=false, socksProxyConfig={}, language=JAVA, namespace=null, mqClientApiTimeout=3000, decodeReadBody=true, decodeDecompressBody=true, enableStreamRequestType=false] success.
[2026-01-20 09:29:19.735] ERROR scheduling-1 - Unexpected error occurred in scheduled task
java.lang.RuntimeException: org.apache.rocketmq.remoting.exception.RemotingSendRequestException: send request to <10.2.0.10:10909> failed
	at org.apache.rocketmq.dashboard.task.DashboardCollectTask.fetchBrokerRuntimeStats(DashboardCollectTask.java:154)
	at org.apache.rocketmq.dashboard.task.DashboardCollectTask.fetchBrokerRuntimeStats(DashboardCollectTask.java:152)
	at org.apache.rocketmq.dashboard.task.DashboardCollectTask.fetchBrokerRuntimeStats(DashboardCollectTask.java:152)
	at org.apache.rocketmq.dashboard.task.DashboardCollectTask.collectBroker(DashboardCollectTask.java:116)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.scheduling.support.ScheduledMethodRunnable.run(ScheduledMethodRunnable.java:84)
	at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:54)
	at org.springframework.scheduling.concurrent.ReschedulingRunnable.run(ReschedulingRunnable.java:95)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.rocketmq.remoting.exception.RemotingSendRequestException: send request to <10.2.0.10:10909> failed
	at org.apache.rocketmq.remoting.netty.NettyRemotingAbstract.invokeSyncImpl(NettyRemotingAbstract.java:495)
	at org.apache.rocketmq.remoting.netty.NettyRemotingClient.invokeSync(NettyRemotingClient.java:565)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.getBrokerRuntimeInfo(MQClientAPIImpl.java:1656)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.fetchBrokerRuntimeStats(DefaultMQAdminExtImpl.java:411)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExt.fetchBrokerRuntimeStats(DefaultMQAdminExt.java:282)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl.fetchBrokerRuntimeStats(MQAdminExtImpl.java:203)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$FastClassBySpringCGLIB$$a15c4ca6.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:783)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:89)
	at org.apache.rocketmq.dashboard.aspect.admin.MQAdminAspect.aroundMQAdminMethod(MQAdminAspect.java:52)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:634)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:624)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:72)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:698)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$EnhancerBySpringCGLIB$$e1758178.fetchBrokerRuntimeStats(<generated>)
	at org.apache.rocketmq.dashboard.task.DashboardCollectTask.fetchBrokerRuntimeStats(DashboardCollectTask.java:142)
	... 17 common frames omitted
[2026-01-20 09:29:31.583] INFO collectTopicThread_1 - create MQAdmin instance ClientConfig [namesrvAddr=154.8.237.182:9876, clientIP=26.26.26.1, instanceName=1768872570573, clientCallbackExecutorThreads=12, pollNameServerInterval=30000, heartbeatBrokerInterval=30000, persistConsumerOffsetInterval=5000, pullTimeDelayMillsWhenException=1000, unitMode=false, unitName=null, vipChannelEnabled=true, useTLS=false, socksProxyConfig={}, language=JAVA, namespace=null, mqClientApiTimeout=3000, decodeReadBody=true, decodeDecompressBody=true, enableStreamRequestType=false] success.
[2026-01-20 09:29:31.594] INFO collectTopicThread_2 - create MQAdmin instance ClientConfig [namesrvAddr=154.8.237.182:9876, clientIP=26.26.26.1, instanceName=1768872570573, clientCallbackExecutorThreads=12, pollNameServerInterval=30000, heartbeatBrokerInterval=30000, persistConsumerOffsetInterval=5000, pullTimeDelayMillsWhenException=1000, unitMode=false, unitName=null, vipChannelEnabled=true, useTLS=false, socksProxyConfig={}, language=JAVA, namespace=null, mqClientApiTimeout=3000, decodeReadBody=true, decodeDecompressBody=true, enableStreamRequestType=false] success.
[2026-01-20 09:29:31.604] INFO collectTopicThread_4 - create MQAdmin instance ClientConfig [namesrvAddr=154.8.237.182:9876, clientIP=26.26.26.1, instanceName=1768872570575, clientCallbackExecutorThreads=12, pollNameServerInterval=30000, heartbeatBrokerInterval=30000, persistConsumerOffsetInterval=5000, pullTimeDelayMillsWhenException=1000, unitMode=false, unitName=null, vipChannelEnabled=true, useTLS=false, socksProxyConfig={}, language=JAVA, namespace=null, mqClientApiTimeout=3000, decodeReadBody=true, decodeDecompressBody=true, enableStreamRequestType=false] success.
[2026-01-20 09:29:35.071] ERROR collectTopicThread_3 - Failed to collect topic: RMQ_SYS_ROCKSDB_TRANS_OP_HALF_TOPIC data
org.apache.rocketmq.remoting.exception.RemotingTimeoutException: wait response on the channel <10.2.0.10:10909> timeout, 4998(ms)
	at org.apache.rocketmq.remoting.netty.NettyRemotingAbstract.invokeSyncImpl(NettyRemotingAbstract.java:493)
	at org.apache.rocketmq.remoting.netty.NettyRemotingClient.invokeSync(NettyRemotingClient.java:565)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.queryTopicConsumeByWho(MQClientAPIImpl.java:2157)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.queryTopicConsumeByWho(DefaultMQAdminExtImpl.java:1050)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExt.queryTopicConsumeByWho(DefaultMQAdminExt.java:494)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl.queryTopicConsumeByWho(MQAdminExtImpl.java:350)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$FastClassBySpringCGLIB$$a15c4ca6.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:783)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:89)
	at org.apache.rocketmq.dashboard.aspect.admin.MQAdminAspect.aroundMQAdminMethod(MQAdminAspect.java:52)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:634)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:624)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:72)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:698)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$EnhancerBySpringCGLIB$$e1758178.queryTopicConsumeByWho(<generated>)
	at org.apache.rocketmq.dashboard.task.CollectTaskRunnble.run(CollectTaskRunnble.java:57)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
[2026-01-20 09:29:36.621] ERROR collectTopicThread_1 - Failed to collect topic: RMQ_SYS_ROCKSDB_TRANS_HALF_TOPIC data
org.apache.rocketmq.remoting.exception.RemotingSendRequestException: send request to <10.2.0.10:10909> failed
	at org.apache.rocketmq.remoting.netty.NettyRemotingAbstract.invokeSyncImpl(NettyRemotingAbstract.java:495)
	at org.apache.rocketmq.remoting.netty.NettyRemotingClient.invokeSync(NettyRemotingClient.java:565)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.queryTopicConsumeByWho(MQClientAPIImpl.java:2157)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.queryTopicConsumeByWho(DefaultMQAdminExtImpl.java:1050)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExt.queryTopicConsumeByWho(DefaultMQAdminExt.java:494)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl.queryTopicConsumeByWho(MQAdminExtImpl.java:350)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$FastClassBySpringCGLIB$$a15c4ca6.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:783)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:89)
	at org.apache.rocketmq.dashboard.aspect.admin.MQAdminAspect.aroundMQAdminMethod(MQAdminAspect.java:52)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:634)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:624)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:72)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:698)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$EnhancerBySpringCGLIB$$e1758178.queryTopicConsumeByWho(<generated>)
	at org.apache.rocketmq.dashboard.task.CollectTaskRunnble.run(CollectTaskRunnble.java:57)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
[2026-01-20 09:29:36.622] ERROR collectTopicThread_2 - Failed to collect topic: TopicTest data
org.apache.rocketmq.remoting.exception.RemotingSendRequestException: send request to <10.2.0.10:10909> failed
	at org.apache.rocketmq.remoting.netty.NettyRemotingAbstract.invokeSyncImpl(NettyRemotingAbstract.java:495)
	at org.apache.rocketmq.remoting.netty.NettyRemotingClient.invokeSync(NettyRemotingClient.java:565)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.queryTopicConsumeByWho(MQClientAPIImpl.java:2157)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.queryTopicConsumeByWho(DefaultMQAdminExtImpl.java:1050)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExt.queryTopicConsumeByWho(DefaultMQAdminExt.java:494)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl.queryTopicConsumeByWho(MQAdminExtImpl.java:350)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$FastClassBySpringCGLIB$$a15c4ca6.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:783)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:89)
	at org.apache.rocketmq.dashboard.aspect.admin.MQAdminAspect.aroundMQAdminMethod(MQAdminAspect.java:52)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:634)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:624)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:72)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:698)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$EnhancerBySpringCGLIB$$e1758178.queryTopicConsumeByWho(<generated>)
	at org.apache.rocketmq.dashboard.task.CollectTaskRunnble.run(CollectTaskRunnble.java:57)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
[2026-01-20 09:29:36.640] ERROR collectTopicThread_4 - Failed to collect topic: broker-a data
org.apache.rocketmq.remoting.exception.RemotingSendRequestException: send request to <10.2.0.10:10909> failed
	at org.apache.rocketmq.remoting.netty.NettyRemotingAbstract.invokeSyncImpl(NettyRemotingAbstract.java:495)
	at org.apache.rocketmq.remoting.netty.NettyRemotingClient.invokeSync(NettyRemotingClient.java:565)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.queryTopicConsumeByWho(MQClientAPIImpl.java:2157)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.queryTopicConsumeByWho(DefaultMQAdminExtImpl.java:1050)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExt.queryTopicConsumeByWho(DefaultMQAdminExt.java:494)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl.queryTopicConsumeByWho(MQAdminExtImpl.java:350)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$FastClassBySpringCGLIB$$a15c4ca6.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:783)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:89)
	at org.apache.rocketmq.dashboard.aspect.admin.MQAdminAspect.aroundMQAdminMethod(MQAdminAspect.java:52)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:634)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:624)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:72)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:97)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:753)
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:698)
	at org.apache.rocketmq.dashboard.service.client.MQAdminExtImpl$$EnhancerBySpringCGLIB$$e1758178.queryTopicConsumeByWho(<generated>)
	at org.apache.rocketmq.dashboard.task.CollectTaskRunnble.run(CollectTaskRunnble.java:57)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```

