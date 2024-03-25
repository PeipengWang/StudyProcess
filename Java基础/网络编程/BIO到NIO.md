# tomcat中的NIO发展
## 前言
Tomcat目前支持BIO（阻塞 I/O）、NIO（非阻塞 I/O）、AIO（异步非阻塞式IO，NIO的升级版）、APR（Apache可移植运行库）模型，本文主要介绍NIO模型，目前NIO模型在各种分布式、通信、Java系统中有广泛应用，如Dubbo、Jetty、Zookeeper等框架中间件中，都使用NIO的方式实现了基础通信组件
## BIO
传统的BIO模型，每个请求都会创建一个线程，当线程向内核发起读取数据申请时，在内核数据没有准备好之前，线程会一直处于等待数据状态，直到内核把数据准备好并返回
在Tomcat中,由Http11Protocol实现阻塞式的Http协议请求，通过传统的ServerSocket的操作，根据传入的参数设置监听端口，如果端口合法且没有被占用则服务监听成功，再通过一个无限循环来监听客户端的连接，如果没有客户端接入，则主线程阻塞在ServerSocket的accept操作上（在Tomcat8、9的版本中已经不支持BIO）
第一次阻塞 connect调用：等待客户端的连接请求，如果没有客户端连接，服务端将一直阻塞等待
第二次阻塞 accept调用：客户端连接后，服务器会等待客户端发送数据，如果客户端没有发送数据，那么服务端将会一直阻塞等待客户端发送数据
在Tomcat中，维护了一个worker线程池来处理socket请求，如果worker线程池没有空闲线程，则Acceptor将会阻塞，所以在有大量请求连接到服务器却不发送消息（占用线程，阻塞与accept的调用）的情况下，会导致服务器压力极大

![在这里插入图片描述](https://img-blog.csdnimg.cn/aa93da1322504ed0b0c3feac82316616.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/297a2091fff546ac9d28a2928dca6b94.png)

## NIO
NIO模型弥补了BIO模型的不足，它基于选择器检测连接（Socket）的就绪状态通知线程处理，从而达到非阻塞的目的，Tomcat NIO基于I/O复用(select/poll/epoll)模型实现，在NIO中有以下几个概念：
Channel
Chnnel是一个通道，网络数据通过Channel读取和写入，通道和流的不同之处在于流是单向的，而通道是双向的
Selector
多路复用器Selector会不断轮询注册在其上的Channel，如果某个Channel上面发生读或者写，就表明这个Channel处于就绪状态，会被Selector选择，通过SelectionKey（通道监听关键字）可以获取就绪Channel的集合，再进行后续IO操作。那么只要有一个线程负责Selector轮询，那么就可以接入成千上万个客户端
在Tomcat中，由NioEndpoint处理非阻塞 IO 的 HTTP/1.1 协议的请求

bind()的作用在于：开启 ServerSocketChannel ，通过ServerSocketChannel 绑定地址、端口
startInternal()主要作用在于初始化连接，启动工作线程池poller 线程组、acceptor 线程组。
acceptor用于监听Socket连接请求，每个acceptor启动以后就开始循环调用 ServerSocketChannel 的 accept() 方法获取新的连接，然后调用 endpoint.setSocketOptions(socket) 处理新的连接，在endpoint.setSocketOptions(socket) 中 则会通过getPoller0().register(channel)，将当前的NioChannel 注册到Poller中，此逻辑在Acceptor .run()中处理

调用getPoller0().register(channel)后，请求socket被包装为一个 PollerEvent，然后添加到 events 中，此过程是由poller线程去做的，poller 的 run() 会循环调用 events() 方法处理注册到 Selector （每一个poller会开启一个 Selector）上的channal ，监听该 channel 的 OP_READ 事件，如果状态为 readable，那么在 processKey ()中将该任务放到 worker 线程池中执行。整个过程大致如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/2f5537ac2d684aebb29418abbc7f8b1b.png)

在NIO模型，不是一个连接就要对应一个处理线程了，连接会被注册到Selector上面，当Selector监听到有效的请求，才会分发一个对应线程去处理，当连接没有请求时，是没有工作线程来处理的
## Tomcat中修改
在Tomcat中指定连接器使用的IO协议，可以通过server.xml的《connector》元素中的protocol属性进行指定，默认值是HTTP/1.1，表明当前版本的默认协议，可以通过把HTTP/1.1修改为以下指定使用的IO协议
org.apache.coyote.http11.Http11Protocol：BIO
org.apache.coyote.http11.Http11NioProtocol：NIO
org.apache.coyote.http11.Http11Nio2Protocol：NIO2
org.apache.coyote.http11.Http11AprProtocol：APR
## 总结
BIO每个连接都会创建一个线程，对性能开销大，不适合高并发场景。
NIO基于多路复用选择器监测连接状态在通知线程处理，当监控到连接上有请求时，才会分配一个线程来处理，利用少量的线程来管理了大量的连接，优化了IO的读写，但同时也增加CPU的计算，适用于连接数较多的场景


