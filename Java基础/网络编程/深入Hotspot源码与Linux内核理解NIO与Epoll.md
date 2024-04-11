转载自[Netty篇-第1章-深入Hotspot源码与Linux内核理解NIO与Epoll](https://zhuanlan.zhihu.com/p/589980018?utm_id=0)

@[TOC](这里写目录标题)
# 一、I/O模型详解
## 1.内核空间和用户空间
思考：物理内存是有限的（比如16G内存），怎么把有限的内存分配给不同的进程？
Linux 给每个进程虚拟出一块很大的地址空间，比如 32 位机器上进程的虚拟内存地址空间是 4GB，从 0x00000000 到 0xFFFFFFFF。但这 4GB 并不是真实的物理内存，而是进程访问到了某个虚拟地址，如果这个地址还没有对应的物理内存页，就会产生缺页中断，分配物理内存，MMU（内存管理单元）会将虚拟地址与物理内存页的映射关系保存在页表中，再次访问这个虚拟地址，就能找到相应的物理内存页。每个进程的这 4GB 虚拟地址空间分布如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e908779eb1ed4c848ba9291e10699594.png)

用户空间从低到高依次是代码区、数据区、堆、共享库与 mmap 内存映射区、栈、环境变量。其中堆向高地址增长，栈向低地址增长。
用户空间上还有一个共享库和 mmap 映射区，Linux 提供了内存映射函数 mmap， 它可将文件内容映射到这个内存区域，用户通过读写这段内存，从而实现对文件的读取和修改，无需通过 read/write 系统调用来读写文件，省去了用户空间和内核空间之间的数据拷贝，Java 的 MappedByteBuffer 就是通过它来实现的；用户程序用到的系统共享库也是通过 mmap 映射到了这个区域。
task_struct结构体本身是分配在内核空间，它的vm_struct成员变量保存了各内存区域的起始和终止地址，此外task_struct中还保存了进程的其他信息，比如进程号、打开的文件、创建的 Socket 以及 CPU 运行上下文
进程的虚拟地址空间总体分为用户空间和内核空间，低地址上的 3GB 属于用户空间，高地址的 1GB 是内核空间，这是基于安全上的考虑，用户程序只能访问用户空间，内核程序可以访问整个进程空间，并且只有内核可以直接访问各种硬件资源，比如磁盘和网卡。

那用户程序需要访问这些硬件资源该怎么办呢？答案是通过系统调用，系统调用可以理解为内核实现的函数，比如应用程序要通过网卡接收数据，会调用 Socket 的 read 函数：
```java
ssize_t read(int fd,void *buf,size_t nbyte)
```
CPU 在执行系统调用的过程中会从用户态切换到内核态，CPU 在用户态下执行用户程序，使用的是用户空间的栈，访问用户空间的内存；当 CPU 切换到内核态后，执行内核代码，使用的是内核空间上的栈。

在 Linux 中，线程是一个轻量级的进程，轻量级说的是线程只是一个 CPU 调度单元，因此线程有自己的task_struct结构体和运行栈区，但是线程的其他资源都是跟父进程共用的，比如虚拟地址空间、打开的文件和 Socket 等。
## 2.阻塞与唤醒
**思考：当用户线程发起一个阻塞式的 read 调用，数据未就绪时，线程就会阻塞，那阻塞具体是如何实现的呢？**
Linux 内核将线程当作一个进程进行 CPU 调度，内核维护了一个可运行的进程队列，所有处于TASK_RUNNING状态的进程都会被放入运行队列中，本质是用双向链表将task_struct链接起来，排队使用 CPU 时间片，时间片用完重新调度 CPU。所谓调度就是在可运行进程列表中选择一个进程，再从 CPU 列表中选择一个可用的 CPU，将进程的上下文恢复到这个 CPU 的寄存器中，然后执行进程上下文指定的下一条指令。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d2d2ce9d18d4ed78cc6dd1c2dce0057.png)
而阻塞的本质就是将进程的task_struct移出运行队列，添加到等待队列，并且将进程的状态的置为TASK_UNINTERRUPTIBLE或者TASK_INTERRUPTIBLE，重新触发一次 CPU 调度让出 CPU。

**思考：线程是如何唤醒的呢？**
线程在加入到等待队列的同时向内核注册了一个回调函数，告诉内核我在等待这个 Socket 上的数据，如果数据到了就唤醒我。这样当网卡接收到数据时，产生硬件中断，内核再通过调用回调函数唤醒进程。唤醒的过程就是将进程的task_struct从等待队列移到运行队列，并且将task_struct的状态置为TASK_RUNNING，这样进程就有机会重新获得 CPU 时间片。
这个过程中，内核还会将数据从内核空间拷贝到用户空间的堆上。
当 read 系统调用返回时，CPU 又从内核态切换到用户态，继续执行 read 调用的下一行代码，并且能从用户空间上的 Buffer 读到数据了。
## 3.Socket Read 系统调用的过程
以Linux操作系统为例，一次socket read 系统调用的过程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/5aff057629a743a3b1d83bede50fdc46.png)
首先 CPU 在用户态执行应用程序的代码，访问进程虚拟地址空间的用户空间；
read 系统调用时 CPU 从用户态切换到内核态，执行内核代码，内核检测到 Socket 上的数据未就绪时，将进程的task_struct结构体从运行队列中移到等待队列，并触发一次 CPU 调度，这时进程会让出 CPU；
当网卡数据到达时，内核将数据从内核空间拷贝到用户空间的 Buffer，接着将进程的task_struct结构体重新移到运行队列，这样进程就有机会重新获得 CPU 时间片，系统调用返回，CPU 又从内核态切换到用户态，访问用户空间的数据。
总结：
当用户线程发起 I/O 调用后，网络数据读取操作会经历两个步骤：
用户线程等待内核将数据从网卡拷贝到内核空间。（数据准备阶段）
内核将数据从内核空间拷贝到用户空间（应用进程的缓冲区）。
各种 I/O 模型的区别就是：
它们实现这两个步骤的方式是不一样的。
## 4.Unix(linux)下5种I/O模型
### 4.1 I/O 模型作用
是为了解决内存和外部设备速度差异的问题。
### 4.2 阻塞或非阻塞
是指应用程序在发起 I/O 操作时，是立即返回（非阻塞）还是等待（阻塞）。
### 4.3 同步和异步
是指应用程序在与内核通信时，数据从内核空间到应用空间的拷贝，是由内核主动发起（异步）还是由应用程序来触发（同步）。
Linux 系统下的 I/O 模型有 5 种：
同步阻塞I/O（bloking I/O）
同步非阻塞I/O（non-blocking I/O）
I/O多路复用（multiplexing I/O）
信号驱动式I/O（signal-driven I/O）
异步I/O（asynchronous I/O）
其中信号驱动式IO在实际中并不常用
![在这里插入图片描述](https://img-blog.csdnimg.cn/72d1131e01a142c4bebb0712c8cde525.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/04d7c15c631a460e9250d6a6af44a89e.png)

## 5.网络IO分类
Java共支持3种网络编程IO模式：BIO，NIO，AIO

# 二、BIO-同步阻塞模型
## 1.概念
同步阻塞模型，一个客户端连接对应一个处理线程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e89c06868fb4ae79dfb2fdb9d712783.png)

## 2.工作逻辑图

## 3.BIO代码逻辑
![在这里插入图片描述](https://img-blog.csdnimg.cn/5cdc09835d9d45398cf2f6437f7a36d6.png)
## 4.BIO代码示例
### 4.1 服务端单线程代码

缺点：多个客户端连接到服务端后，读取客户端的数据是单线程阻塞的，需要按顺序逐个处理，非常耗时。

```java
package com.tuling.bio;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(9000);
        while (true) {
            System.out.println("等待连接。。");
            //阻塞方法
            Socket clientSocket = serverSocket.accept();
            System.out.println("有客户端连接了。。");
            (1)单个线程，多个客户端连接，后面的客户端连接发送信息，
            无法立即到服务端。需要阻塞等待前面的客户端处理完成后才能
            继续处理后面的客户端请求
            handler(clientSocket);

        }
    }

    private static void handler(Socket clientSocket) throws IOException {
        byte[] bytes = new byte[1024];
        System.out.println("准备read。。");
        //接收客户端的数据，阻塞方法，没有数据可读时就阻塞
        int read = clientSocket.getInputStream().read(bytes);
        System.out.println("read完毕。。");
        if (read != -1) {
            System.out.println("接收到客户端的数据：" + new String(bytes, 0, read));
        }
        clientSocket.getOutputStream().write("HelloClient".getBytes());
        clientSocket.getOutputStream().flush();
    }
}
```

### 4.2 服务端多线程代码

```java
package com.tuling.bio;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(9000);
        while (true) {
            System.out.println("等待连接。。");
            //阻塞方法
            Socket clientSocket = serverSocket.accept();
            System.out.println("有客户端连接了。。");    
                               
        (2)多线程客户端连接
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        handler(clientSocket);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }

    private static void handler(Socket clientSocket) throws IOException {
        byte[] bytes = new byte[1024];
        System.out.println("准备read。。");
        //接收客户端的数据，阻塞方法，没有数据可读时就阻塞
        int read = clientSocket.getInputStream().read(bytes);
        System.out.println("read完毕。。");
        if (read != -1) {
            System.out.println("接收到客户端的数据：" + new String(bytes, 0, read));
        }
        clientSocket.getOutputStream().write("HelloClient".getBytes());
        clientSocket.getOutputStream().flush();
    }
}
```

### 4.3 客户端代码

```java
//客户端代码
public class SocketClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", 9000);
        //向服务端发送数据
        socket.getOutputStream().write("HelloServer".getBytes());
        socket.getOutputStream().flush();
        System.out.println("向服务端发送数据结束");
        byte[] bytes = new byte[1024];
        //接收服务端回传的数据
        socket.getInputStream().read(bytes);
        System.out.println("接收到服务端的数据：" + new String(bytes));
        socket.close();
    }
}
```


## 5.缺点
1、单线程情况下：IO代码里read操作是阻塞操作，如果连接不做数据读写操作会导致线程阻塞，浪费资源
2、多线程情况下：会导致服务器线程太多，压力太大，比如C10K问题(10000个线程连接，导致服务资源占满)
## 6.应用场景
BIO 方式适用于连接数目比较小且固定的架构， 这种方式对服务器资源要求比较高， 但程序简单易理解。

## 7.telnet连接服务端测试
CMD任务框：

![在这里插入图片描述](https://img-blog.csdnimg.cn/18c1639c5272490bb6b76bc13d24fe99.png)

输入命令telnet

#连接服务端，端口是9000，按回车--》按'CTRL+] '
```c
 telnet localhost 9000
```
发送数据到服务端

```c
send  测试数据
```

# 三、NIO-同步非阻塞
## 1.概念
同步非阻塞，服务端实现模式为一个线程可以处理多个请求(连接)。

客户端发送的连接请求都会注册到多路复用器selector上，多路复用器轮询到连接有IO请求就进行处理，JDK1.4开始引入。

## 2.应用场景
NIO方式适用于连接数目多且连接比较短（轻操作） 的架构， 比如聊天服务器， 弹幕系统， 服务器间通讯，编程比较复杂

## 3.服务端非阻塞实现
### 3.1 轮询SocketChannel
```java
package com.tuling.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class NioServer {

    // 保存客户端连接
    static List<SocketChannel> channelList = new ArrayList<>();

    public static void main(String[] args) throws IOException, InterruptedException {

        // 创建NIO ServerSocketChannel,与BIO的serverSocket类似
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.socket().bind(new InetSocketAddress(9000));
        // 设置ServerSocketChannel为非阻塞
        serverSocket.configureBlocking(false);
        System.out.println("服务启动成功");

        while (true) {
            // 非阻塞模式accept方法不会阻塞，否则会阻塞
            // NIO的非阻塞是由操作系统内部实现的，底层调用了linux内核的accept函数
            SocketChannel socketChannel = serverSocket.accept();
            if (socketChannel != null) { // 如果有客户端进行连接
                System.out.println("连接成功");
                // 设置SocketChannel为非阻塞
                socketChannel.configureBlocking(false);
                // 保存客户端连接在List中
                channelList.add(socketChannel);
            }
            // 遍历连接进行数据读取
            Iterator<SocketChannel> iterator = channelList.iterator();
            while (iterator.hasNext()) {
                SocketChannel sc = iterator.next();
                ByteBuffer byteBuffer = ByteBuffer.allocate(128);
                // 非阻塞模式read方法不会阻塞，否则会阻塞
                int len = sc.read(byteBuffer);
                // 如果有数据，把数据打印出来
                if (len > 0) {
                    System.out.println("接收到消息：" + new String(byteBuffer.array()));
                } else if (len == -1) { // 如果客户端断开，把socket从集合中去掉
                    iterator.remove();
                    System.out.println("客户端断开连接");
                }
            }
        }
    }
}
```


总结：如果连接数太多的话，会有大量的无效遍历，假如有10000个连接，其中只有1000个连接有写数据，但是由于其他9000个连接并没有断开，我们还是要每次轮询遍历一万次，其中有十分之九的遍历都是无效的，这显然不是一个让人很满意的状态。这就是C10K问题。



### 3.2 多路复用器Selector

```java
package com.tuling.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NioSelectorServer {

    public static void main(String[] args) throws IOException, InterruptedException {

        // 创建NIO ServerSocketChannel
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        serverSocket.socket().bind(new InetSocketAddress(9000));
        // 设置ServerSocketChannel为非阻塞
        serverSocket.configureBlocking(false);
        // 打开Selector处理Channel，即创建epoll
        Selector selector = Selector.open();
        // 把ServerSocketChannel注册到selector上，并且selector对客户端accept连接操作感兴趣
        serverSocket.register(selector,2222 SelectionKey.OP_ACCEPT);
        System.out.println("服务启动成功");

        while (true) {
            // 阻塞等待需要处理的事件发生
            selector.select();

            // 获取selector中注册的全部事件的 SelectionKey 实例
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();

            // 遍历SelectionKey对事件进行处理
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                // 如果是OP_ACCEPT事件，则进行连接获取和事件注册
                if (key.isAcceptable()) {
                    ServerSocketChannel server = (ServerSocketChannel) key.channel();
                    SocketChannel socketChannel = server.accept();
                    socketChannel.configureBlocking(false);
                    // 这里只注册了读事件，如果需要给客户端发送数据可以注册写事件
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    System.out.println("客户端连接成功");
                } else if (key.isReadable()) {  // 如果是OP_READ事件，则进行读取和打印
                    SocketChannel socketChannel = (SocketChannel) key.channel();
                    ByteBuffer byteBuffer = ByteBuffer.allocate(128);
                    int len = socketChannel.read(byteBuffer);
                    // 如果有数据，把数据打印出来
                    if (len > 0) {
                        System.out.println("接收到消息：" + new String(byteBuffer.array()));
                    } else if (len == -1) { // 如果客户端断开连接，关闭Socket
                        System.out.println("客户端断开连接");
                        socketChannel.close();
                    }
                }
                //从事件集合里删除本次处理的key，防止下次select重复处理
                iterator.remove();
            }
        }
    }
}
```

## 4.NIO 有三大核心组件
Channel(通道)， Buffer(缓冲区)，Selector(多路复用器)
1、channel 类似于流，每个 channel 对应一个 buffer缓冲区，buffer 底层就是个数组
2、channel 会注册到 selector 上，由 selector 根据 channel 读写事件的发生将其交由某个空闲的线程处理
3、NIO 的 Buffer 和 channel 都是既可以读也可以写
## 5、多路复用器Selector模式工作原理图
![在这里插入图片描述](https://img-blog.csdnimg.cn/0cc60a58d67e4b129d675e59fb36e15a.png)
## 6.NIO非阻塞底层原理
（1）轮询机制：在JDK1.4版本是用linux的内核函数select()或poll()来实现，跟上面的NioServer代码类似，selector每次都会轮询所有的sockchannel看下哪个channel有读写事件，有的话就处理，没有就继续遍历。
（2）事件通知机制：JDK1.5开始引入了epoll基于事件响应机制来优化NIO。

## 7.重要方法

```java
Selector.open()  //创建多路复用器
socketChannel.register(selector, SelectionKey.OP_READ) //将channel注册到多路复用器上
selector.select()  //阻塞等待需要处理的事件发生
```

## 8.多路复用器Selector底层源码
![在这里插入图片描述](https://img-blog.csdnimg.cn/8aced36c6db54324bacb27b15fa763dd.png)

NIO整个调用流程：
1.Selector.open() :创建EPoll实例（Selector），用文件描述符epfd代表。创建一个集合EPollArrayWapper。
2.socketChannel.register(selector, SelectionKey.OP_READ):给服务端的socketChannel新建一个文件描述符fd,并将fd加入集合EPollArrayWapper。
3.selector.select()：
3.1:从EPollArrayWapper集合中拿出fd,并绑定将事件与fd,epfd,opcode绑定。
3.2:等待事件触发。将事件处理交给了操作系统内核(操作系统中断程序实现)。
## 9.Epoll函数详解
### 9.1 epoll_create

```java
int epoll_create(int size);
```
创建一个epoll实例，并返回一个非负数作为文件描述符，用于对epoll接口的所有后续调用。参数size代表可能会容纳size个描述符，但size不是一个最大值，只是提示操作系统它的数量级，现在这个参数基本上已经弃用了。

### 9.2 epoll_ctl

```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```
使用文件描述符epfd引用的epoll实例，对目标文件描述符fd执行op操作。
参数epfd表示epoll对应的文件描述符，参数fd表示socket对应的文件描述符。
参数op有以下几个值：
EPOLL_CTL_ADD：注册新的fd到epfd中，并关联事件event；
EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
EPOLL_CTL_DEL：从epfd中移除fd，并且忽略掉绑定的event，这时event可以为null；
参数event是一个结构体

struct epoll_event {
    __uint32_t   events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};

typedef union epoll_data {
    void        *ptr;
    int          fd;
    __uint32_t   u32;
    __uint64_t   u64;
} epoll_data_t;

events有很多可选值，这里只举例最常见的几个：
EPOLLIN ：表示对应的文件描述符是可读的；
EPOLLOUT：表示对应的文件描述符是可写的；
EPOLLERR：表示对应的文件描述符发生了错误；
成功则返回0，失败返回-1
### 9.3 epoll_wait

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);

等待文件描述符epfd上的事件。

epfd是Epoll对应的文件描述符，events表示调用者所有可用事件的集合，maxevents表示最多等到多少个事件就返回，timeout是超时时间。



## 10.Linux 内核函数比较
I/O多路复用底层主要用的Linux 内核·函数（select，poll，epoll）来实现，windows不支持epoll实现，windows底层是基于winsock2的select函数实现的(不开源)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2891f80206944fd0b979ac1f97312d8c.png)

## 11.Redis线程模型
Redis就是典型的基于epoll的NIO线程模型(nginx也是)，epoll实例收集所有事件(连接与读写事件)，由一个服务端线程连续处理所有事件命令。
Redis底层关于epoll的源码实现在redis的src源码目录的ae_epoll.c文件里，感兴趣可以自行研究。

# 三、AIO-异步非阻塞
1.概念
异步非阻塞， 由操作系统完成后回调通知服务端程序启动线程去处理。 一般适用于连接数较多且连接时间较长的应用。
2.应用场景
AIO方式适用于连接数目多且连接比较长(重操作)的架构，JDK7 开始支持
3.AIO代码示例

```java
package com.tuling.aio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;

public class AIOServer {

public static void main(String[] args) throws Exception {
final AsynchronousServerSocketChannel serverChannel =
        AsynchronousServerSocketChannel.open().bind(new InetSocketAddress(9000));

serverChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel, Object>() {
    @Override
    public void completed(AsynchronousSocketChannel socketChannel, Object attachment) {
        try {
    System.out.println("2--"+Thread.currentThread().getName());
    // 再此接收客户端连接，如果不写这行代码后面的客户端连接连不上服务端
    serverChannel.accept(attachment, this);
    System.out.println(socketChannel.getRemoteAddress());
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    socketChannel.read(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {
        @Override
        public void completed(Integer result, ByteBuffer buffer) {
            System.out.println("3--"+Thread.currentThread().getName());
            buffer.flip();
            System.out.println(new String(buffer.array(), 0, result));
            socketChannel.write(ByteBuffer.wrap("HelloClient".getBytes()));
        }

        @Override
        public void failed(Throwable exc, ByteBuffer buffer) {
            exc.printStackTrace();
        }
    });
    } catch (IOException e) {
        e.printStackTrace();
    }
}

    @Override
    public void failed(Throwable exc, Object attachment) {
        exc.printStackTrace();
    }
});

System.out.println("1--"+Thread.currentThread().getName());
Thread.sleep(Integer.MAX_VALUE);
}
}


package com.tuling.aio;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;

public class AIOClient {

    public static void main(String... args) throws Exception {
        AsynchronousSocketChannel socketChannel = AsynchronousSocketChannel.open();
        socketChannel.connect(new InetSocketAddress("127.0.0.1", 9000)).get();
        socketChannel.write(ByteBuffer.wrap("HelloServer".getBytes()));
        ByteBuffer buffer = ByteBuffer.allocate(512);
        Integer len = socketChannel.read(buffer).get();
        if (len != -1) {
            System.out.println("客户端收到信息：" + new String(buffer.array(), 0, len));
        }
    }
}
```

# 四、BIO、 NIO、 AIO 对比
![在这里插入图片描述](https://img-blog.csdnimg.cn/6952875ff9c1427ea6a7812e5a855d27.png)

# 五、为什么Netty使用NIO而不是AIO？
在Linux系统上，AIO的底层实现仍使用Epoll，没有很好实现AIO，因此在性能上没有明显的优势，而且被JDK封装了一层不容易深度优化，Linux上AIO还不够成熟。Netty是异步非阻塞框架，Netty在NIO上做了很多异步的封装。



# 六、同步异步与阻塞非阻塞(段子)
老张爱喝茶，废话不说，煮开水。

出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。

1 老张把水壶放到火上，立等水开。（同步阻塞）

老张觉得自己有点傻

2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞）

老张还是觉得自己有点傻，于是变高端了，买了把会响笛的那种水壶。水开之后，能大声发出嘀~~~~的噪音。

3 老张把响水壶放到火上，立等水开。（异步阻塞）

老张觉得这样傻等意义不大

4 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）

老张觉得自己聪明了。



所谓同步异步，只是对于水壶而言。

普通水壶，同步；响水壶，异步。

虽然都能干活，但响水壶可以在自己完工之后，提示老张水开了。这是普通水壶所不能及的。

同步只能让调用者去轮询自己（情况2中），造成老张效率的低下。

所谓阻塞非阻塞，仅仅对于老张而言。

立等的老张，阻塞；看电视的老张，非阻塞。



# 七、Tomcat的 I/O 模型
1.Tomcat支持I/O模型
Tomcat 支持的多种 I/O 模型和应用层协议。Tomcat 支持的 I/O 模型有：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d0bb8cc57482417982befbf1941f62b3.png)

IO模型	描述
BIO （JIoEndpoint）	同步阻塞式IO，即Tomcat使用传统的http://java.io进行操作。该模式下每个请求都会创建一个线程，对性能开销大，不适合高并发场景。优点是稳定，适合连接数目小且固定架构。
NIO（NioEndpoint）	同步非阻塞式IO，jdk1.4 之后实现的新IO。该模式基于多路复用选择器监测连接状态再同步通知线程处理，从而达到非阻塞的目的。比传统BIO能更好的支持并发性能。Tomcat 8.0之后默认采用该模式。NIO方式适用于连接数目多且连接比较短（轻操作） 的架构， 比如聊天服务器， 弹幕系统， 服务器间通讯，编程比较复杂
AIO (Nio2Endpoint)	异步非阻塞式IO，jdk1.7后之支持 。与nio不同在于不需要多路复用选择器，而是请求处理线程执行完成进行回调通知，继续执行后续操作。Tomcat 8之后支持。一般适用于连接数较多且连接时间较长的应用
APR（AprEndpoint）	全称是 Apache Portable Runtime/Apache可移植运行库)，是Apache HTTP服务器的支持库。AprEndpoint 是通过 JNI 调用 APR 本地库而实现非阻塞 I/O 的。使用需要编译安装APR 库
注意： Linux 内核没有很完善地支持异步 I/O 模型，因此 JVM 并没有采用原生的 Linux 异步 I/O，而是在应用层面通过 epoll 模拟了异步 I/O 模型。因此在 Linux 平台上，Java NIO 和 Java NIO.2 底层都是通过 epoll 来实现的，但是 Java NIO 更加简单高效。

2.Tomcat I/O 模型的选择
I/O 调优实际上是连接器类型的选择，一般情况下默认都是 NIO，在绝大多数情况下都是够用的，除非你的 Web 应用用到了 TLS 加密传输，而且对性能要求极高，这个时候可以考虑 APR，因为 APR 通过 OpenSSL 来处理 TLS 握手和加密 / 解密。OpenSSL 本身用 C 语言实现，它还对 TLS 通信做了优化，所以性能比 Java 要高。如果你的 Tomcat 跑在 Windows 平台上，并且 HTTP 请求的数据量比较大，可以考虑 NIO.2，这是因为 Windows 从操作系统层面实现了真正意义上的异步 I/O，如果传输的数据量比较大，异步 I/O 的效果就能显现出来。



3.指定IO模型只需修改server.xml文件的protocol配置

```java
<!-- 修改protocol属性, 使用NIO2 --> <Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol"            connectionTimeout="20000"            redirectPort="8443" />
```
4.NioEndpoint -同步非阻塞
思考：Tomcat是如何实现非阻塞I/O的？

在 Tomcat 中，EndPoint 组件的主要工作就是处理 I/O，而 NioEndpoint 利用 Java NIO API 实现了多路复用 I/O 模型。Tomcat的NioEndpoint 是基于主从Reactor多线程模型设计。

![在这里插入图片描述](https://img-blog.csdnimg.cn/336e05465898427faadf52b27ab959f5.png)







4.1 NioEndpoint的设计思路



![在这里插入图片描述](https://img-blog.csdnimg.cn/dba0c02dd03f414daa432660e62c6fd4.png)



LimitLatch 是连接控制器，它负责控制最大连接数，NIO 模式下默认是 10000(tomcat9中8192)，当连接数到达最大时阻塞线程，直到后续组件处理完一个连接后将连接数减 1。注意到达最大连接数后操作系统底层还是会接收客户端连接，但用户层已经不再接收。
Acceptor 跑在一个单独的线程里，它在一个死循环里调用 accept 方法来接收新连接，一旦有新的连接请求到来，accept 方法返回一个 Channel 对象，接着把 Channel 对象交给 Poller 去处理。


#NioEndpoint#initServerSocket

serverSock = ServerSocketChannel.open();
//第2个参数表示操作系统的等待队列长度，默认100
//当应用层面的连接数到达最大值时，操作系统可以继续接收的最大连接数
serverSock.bind(addr, getAcceptCount());
//ServerSocketChannel 被设置成阻塞模式
serverSock.configureBlocking(true);
ServerSocketChannel 通过 accept() 接受新的连接，accept() 方法返回获得 SocketChannel 对象，然后将 SocketChannel 对象封装在一个 PollerEvent 对象中，并将 PollerEvent 对象压入 Poller 的 SynchronizedQueue 里，这是个典型的生产者 - 消费者模式，Acceptor 与 Poller 线程之间通过 SynchronizedQueue 通信。



Poller 的本质是一个 Selector，也跑在单独线程里。Poller 在内部维护一个 Channel 数组，它在一个死循环里不断检测 Channel 的数据就绪状态，一旦有 Channel 可读，就生成一个 SocketProcessor 任务对象扔给 Executor 去处理。

Executor 就是线程池，负责运行 SocketProcessor 任务类，SocketProcessor 的 run 方法会调用 Http11Processor 来读取和解析请求数据。Http11Processor 是应用层协议的封装，它会调用容器获得响应，再把响应通过 Channel 写出。






4.2 设计精髓：Tomcat线程池扩展

思考：Tomcat是如何扩展java线程池的？

Tomcat线程池默认实现StandardThreadExecutor。Tomcat 线程池和 Java 原生线程池的区别：



自定义了拒绝策略，Tomcat 在线程总数达到最大数时，不是立即执行拒绝策略，而是再尝试向任务队列添加任务，添加失败后再执行拒绝策略。
TaskQueue 重写了 LinkedBlockingQueue 的 offer 方法。只有当前线程数大于核心线程数、小于最大线程数，并且已提交的任务个数大于当前线程数时，也就是说线程不够用了，但是线程数又没达到极限，才会去创建新的线程。目的：在任务队列的长度无限制的情况下，让线程池有机会创建新的线程。




4.3 设计精髓：NIO中涉及的对象池技术

Java 对象，特别是一个比较大、比较复杂的 Java 对象，它们的创建、初始化和 GC 都需要耗费 CPU 和内存资源，为了减少这些开销，Tomcat 使用了对象池技术。对象池技术可以减少频繁创建和销毁对象带来的成本，实现对象的缓存和复用，是典型的以空间换时间的设计思路。

思考： PollerEvent为什么采用SynchronizedStack缓存？

/** * Cache for poller events */ private SynchronizedStack<PollerEvent> eventCache;

SynchronizedStack 内部维护了一个对象数组，并且用数组来实现栈的接口：push 和 pop 方法，这两个方法分别用来归还对象和获取对象。SynchronizedStack 用数组而不是链表来维护对象，可以减少结点维护的内存开销，并且它本身只支持扩容不支持缩容，也就是说数组对象在使用过程中不会被重新赋值，也就不会被 GC。这样设计的目的是用最低的内存和 GC 的代价来实现无界容器，同时 Tomcat 的最大同时请求数是有限制的，因此不需要担心对象的数量会无限膨胀。



5.Nio2Endpoint-异步非阻塞
NIO 和 NIO.2 最大的区别是，一个是同步一个是异步。异步最大的特点是，应用程序不需要自己去触发数据从内核空间到用户空间的拷贝。

思考：Tomcat如何实现异步I/O的？
![在这里插入图片描述](https://img-blog.csdnimg.cn/f15eca7755814981a39c2694eff803ae.png)


Nio2Endpoint 中没有 Poller 组件，也就是没有 Selector。在异步 I/O 模式下，Selector 的工作交给内核来做了。




