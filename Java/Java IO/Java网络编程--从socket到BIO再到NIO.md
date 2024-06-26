# Java网络编程原理与实践--从Socket到BIO再到NIO
## Socket基本架构
图来源：https://zhuanlan.zhihu.com/p/462497498
既然是网络的东西肯定得放个网络架构图，这张图不多说，感兴趣可以去链接详细看一下
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/49f33f5f19e7439bbdad1d164f853790.png)

## Socket 基本使用
转自：https://blog.csdn.net/a78270528/article/details/80318571
### 简单一次发送接收
#### 客户端
```
package Scoket.client;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

/***
 * 字符流方式
 */
public class Client {
    public static void main(String[] args) {
        try {
            // 服务器的主机和端口
            String serverHost = "127.0.0.1";
            int serverPort = 6443;

            // 创建Socket对象，连接到服务器
            Socket socket = new Socket(serverHost, serverPort);

            // 获取输入流和输出流
            BufferedReader input = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter output = new PrintWriter(socket.getOutputStream(), true);

            // 发送数据给服务器
            String messageToSend = "Hello, Server!";
            output.println(messageToSend);

            // 接收服务器的响应数据
            String dataReceived = input.readLine();
            System.out.println("Received from server: " + dataReceived);

            // 关闭连接
            socket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```
#### 服务端
```
package Scoket.client;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) {
        try {
            // 本地主机和端口
            String serverHost = "127.0.0.1";
            int serverPort = 6443;
            // 创建ServerSocket对象，绑定地址和端口
            ServerSocket serverSocket = new ServerSocket(serverPort);
            System.out.println("Server listening on " + serverHost + ":" + serverPort);
            // 接受客户端连接
            Socket clientSocket = serverSocket.accept();
            System.out.println("Accepted connection from " + clientSocket.getInetAddress());
            // 获取输入流和输出流
            BufferedReader input = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
            PrintWriter output = new PrintWriter(clientSocket.getOutputStream(), true);
            // 接收客户端发送的数据
            String dataReceived = input.readLine();
            System.out.println("Received from client: " + dataReceived);
            // 发送响应给客户端
            String messageToSend = "Hello, Client!";
            output.println(messageToSend);
            // 关闭连接
            clientSocket.close();
            serverSocket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
如果进行debug会发现，服务端代码总共卡主两次：
1、  Socket clientSocket = serverSocket.accept();  这里会监听端口，等待客户端请求建立连接，实际上是进行三次握手  
2、 String dataReceived = input.readLine();    这里是等待客户端发送数据，接收到数据会进行下一步  
这两步骤需要注意，因为这是后面BIO和NIO的优化点  
### 字节流方式简单发送接收
使用字节流处理，这可能使得处理字符串数据稍显繁琐。如果你的通信数据是文本，可能使用字符流更为方便。  
但是数据更可控一些，下面简单罗列  
#### 客户端
```
package Scoket.client;

import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

/**
 * 字节流方式
 */
public class Client1 {
    public static void main(String[] args) {
        try {
            String host = "127.0.0.1";
            int port = 6443;
            Socket socket = new Socket(host, port);
            OutputStream outputStream = socket.getOutputStream();
            String message = "message， 你好";
            socket.getOutputStream().write(message.getBytes(StandardCharsets.UTF_8));
            outputStream.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }


    }
}

```
#### 服务端
```
package Scoket.client;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class Server1 {
    public static void main(String[] args) {
        try {
            int port = 6443;
            ServerSocket serverSocket = new ServerSocket(port);
            System.out.println("等待连接");
            Socket accept = serverSocket.accept();
            System.out.println("完成连接，等待传输数据");
            InputStream inputStream = accept.getInputStream();
            byte[] bytes = new byte[1024];
            int len;
            StringBuilder sb = new StringBuilder();
            while ((len = inputStream.read(bytes)) != -1){
                sb.append(new String(bytes, 0, len, "UTF-8"));
            }
            System.out.println("get message:" + sb);
            inputStream.close();
            accept.close();
            serverSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 双向通信
#### 客户端
```
package Scoket.client;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class Client2 {
    public static void main(String[] args) {
        try {
            String host = "127.0.0.1";
            int port = 8443;
            Socket socket = new Socket(host, port);
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write("我是客户，接受一下我的消息".getBytes(StandardCharsets.UTF_8));
            socket.shutdownOutput();
            InputStream inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            int len;
            StringBuilder stringBuilder = new StringBuilder();
            while ((len = inputStream.read(bytes)) != -1){
                stringBuilder.append(new String(bytes, 0, len, "UTF-8"));
            }
            System.out.println("get message:" + stringBuilder);
            inputStream.close();
            outputStream.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
#### 服务端
```
package Scoket.client;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class Server2 {
    public static void main(String[] args) {
        try {
            int port = 8443;
            ServerSocket serverSocket = new ServerSocket(port);
            Socket socket = serverSocket.accept();
            InputStream inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            int len ;
            StringBuilder sb = new StringBuilder();
            while ((len = inputStream.read(bytes)) != -1){
                sb.append(new String(bytes, 0, len, "UTF-8"));
            }
            System.out.println("server2 get Message:" + sb);
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write("我是服务器".getBytes(StandardCharsets.UTF_8));
            inputStream.close();
            outputStream.close();
            socket.close();
            serverSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 多次接收消息
#### 客户端
```
package Scoket.client;

import java.io.OutputStream;
import java.net.Socket;
 
public class Client3 {
  public static void main(String args[]) throws Exception {
    // 要连接的服务端IP地址和端口
    String host = "127.0.0.1";
    int port = 8444;
    // 与服务端建立连接
    Socket socket = new Socket(host, port);
    // 建立连接后获得输出流
    OutputStream outputStream = socket.getOutputStream();
    String message = "你好  yiwangzhibujian";
    //首先需要计算得知消息的长度
    byte[] sendBytes = message.getBytes("UTF-8");
    //然后将消息的长度优先发送出去
    outputStream.write(sendBytes.length >>8);
    outputStream.write(sendBytes.length);
    //然后将消息再次发送出去
    outputStream.write(sendBytes);
    outputStream.flush();
    //==========此处重复发送一次，实际项目中为多个命名，此处只为展示用法
    message = "第二条消息";
    sendBytes = message.getBytes("UTF-8");
    outputStream.write(sendBytes.length >>8);
    outputStream.write(sendBytes.length);
    outputStream.write(sendBytes);
    outputStream.flush();
    //==========此处重复发送一次，实际项目中为多个命名，此处只为展示用法
    message = "the third message!";
    sendBytes = message.getBytes("UTF-8");
    outputStream.write(sendBytes.length >>8);
    outputStream.write(sendBytes.length);
    outputStream.write(sendBytes);    
    
    outputStream.close();
    socket.close();
  }
}
```
#### 服务端
```
package Scoket.client;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class Server3 {
    public static void main(String[] args) {
        try {
            int port = 8444;
            ServerSocket serverSocket = new ServerSocket(port);
            Socket accept = serverSocket.accept();
            InputStream inputStream = accept.getInputStream();
            byte[] bytes;
            while (true){
                int first = inputStream.read();
                if(first == -1){
                    break;
                }
                int second = inputStream.read();
                int len = (first << 8) +second;
                bytes = new byte[len];
                inputStream.read(bytes);
                System.out.println("Server3 get message:" + new String(bytes, "UTF-8"));

            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```
## Socket写法的问题
上面的代码有些很大的问题  
1、阻塞式 I/O： 这是最大的缺点之一。在 accept()、readLine() 等方法调用时，程序会被阻塞，等待客户端连接或数据到来。这可能导致服务器在处理多个客户端时性能下降。  
2、单线程处理： 服务器采用单线程处理客户端连接。这意味着一次只能处理一个客户端连接，如果有大量的客户端同时连接，性能会受到影响。  
3、不适用于高并发： 由于采用单线程处理方式，不适合高并发环境。在高并发情况下，建议考虑使用多线程或异步 I/O 模型。  
4、异常处理不足： 缺少一些异常处理，例如，在 accept()、readLine() 中可能会抛出异常，而在示例中并未捕获和处理这些异常。  
针对1、2可以采用BIO方式  
针对1、2、3可以采用NIO  
接下来将会优化代码分别介绍BIO和NIO  
## BIO
### 简单流程  
服务器启动一个ServerSocket。  
客户端启动一个Socket对服务器进行通信，默认情况下，服务器端需要对每一个客户端建立一个线程与之通信。  
客户端发出请求后，先咨询服务器是否有线程相应，如果没有则会等待，或者被拒绝。  
如果有响应，客户端线程会等待请求结束后，再继续执行。  

### BIO写法
#### 客户端
```
package Scoket.client;

import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class BIOClient{
    public static void main(String[] args) {
        try {
            Socket socket = new Socket("127.0.0.1", 6666);
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write("hi, i am client".getBytes(StandardCharsets.UTF_8));
            outputStream.flush();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }


    }
}
```
#### 服务端
转自：https://juejin.cn/post/6924670437867651080
```
package Scoket.client;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class BIOServer {

    public static void main(String[] args) throws IOException {
        // 创建线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        // 创建ServerSocket并且监听6666端口
        ServerSocket serverSocket = new ServerSocket(6666);
        while (true) {
            // 监听---一直等待客户端连接
            Socket socket = serverSocket.accept();
            // 连接来了之后，启用一个线程去执行里面的方法
            executorService.execute(() -> {
                try {
                    // 获取客户端发送过来的输入流
                    InputStream inputStream = socket.getInputStream();
                    byte[] bytes = new byte[1024];
                    int read = inputStream.read(bytes);
                    // 读取发送过来的信息并打印
                    if (read != -1) {
                        System.out.println(new String(bytes, 0, read));
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    // 断开通讯
                    try {
                        socket.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}
```
### BIO的问题
上述写法主要是在服务端接受到一个客户端连接时，就开启一个线程，然后新建一个连接专门处理这个服务  
可以看下accept代码  
```
    public Socket accept() throws IOException {
        if (this.isClosed()) {
            throw new SocketException("Socket is closed");
        } else if (!this.isBound()) {
            throw new SocketException("Socket is not bound yet");
        } else {
            Socket s = new Socket((SocketImpl)null);
            this.implAccept(s);
            return s;
        }
    }

```
可以看到，每次accept就会新建一个Socket  
因此会有如下问题：  
每个请求都需要创建独立的线程，与对应的客户端进行数据读，业务处理，然后再数据写。  
当并发数较大时，需要创建大量的线程来处理连接，系统资源占用较大。  
连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在读操作上，造成线程资源浪费。  

基于上面的问题产生了NIO  

## NIO
### 简述
Java NIO全称java non-blocking IO，是指JDK提供的新API。从JDK1.4开始，提供了一系列改进的输入/输出的新特性，被统称为NIO（所以也可称为New IO），是同步非阻塞的。  
NIO相关类都被放在java.nio包及子包下，并且对原java.io包中的很多类进行改写。  
NIO有三大核心部分：  
Channel（通道）  
Buffer（缓冲区）  
Selector（选择器）  
NIO是面向缓冲区的。数据读取到一个它的稍后处理的缓冲区，需要时可以在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络。  
Java NIO的非阻塞模式，是一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变得可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。  
通俗理解：NIO是可以做到用一个线程来处理多个操作的。假设有10000个请求过来，根据实际情况，可以分配50或者100个线程来处理。不像之前的阻塞IO那样，非得分配10000个。  
HTTP2.0使用了多路复用的技术，做到了同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1大了好几个数量级。  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/e69f7d1d1c88438eab439dcf7c4df3df.png)


### Buffer
Buffer（缓冲区）：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象（含数组），该对象提供了一组方法，可以更轻松的使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer。  
在使用Buffer进行数据读写的时候，主要是通过底层的这个数组来储存数据，但是具体的控制数据读写，是通过父类Buffer中的以下参数来控制的：  

|   属性    |                                         描述                                          |
| -------- | ------------------------------------------------------------------------------------ |
| Capacity | 容量，即可以容纳的最大数据量。在缓冲区被创建时被确定并且不能改变                            |
| Limit    | 示缓冲区的当前终点，不能对缓冲区超过limit的位置进行读写操作，且limit是可以修改的             |
| Position | 位置，下一个要被读/写的元素的索引，每次读写缓冲区数据时都会改变position的值，为下次读写做准备 |
| Mark     | 标记                                                                                  |

一共有7个类直接继承了Buffer类，这7个子类分别是除了boolean外的其他7中数据类型的Buffer类。  
在这七个子类中，都有一个相应数据类型的数组，比如IntBuffer中就有一个int类型的数组：  
final int[] hb;    
在ByteBuffer类中就有一个byte类型的数组：  
final byte[] hb;   
实例：
```
package Scoket.client;

import java.nio.IntBuffer;

public class Buffer {
    public static void main(String[] args) {
        // 创建一个IntBuffer对象实例，分配容量为5
        IntBuffer buffer = IntBuffer.allocate(5);
        for (int i = 0; i < buffer.capacity(); i++) {
            // 每次循环为buffer塞一个int类型的数值，经过5次循环后，buffer中应该有0、2、4、6、8这5个数
            buffer.put(i * 2);
        }
        // 当要将buffer从写入转换到读取的时候，需要调用flip()方法
        // flip()方法是将limit指向position的位置，并且再将position置0
        // 表示从头再读到调用flip()方法的地方
        buffer.flip();
        // hasRemaining()方法表示是否还有剩余的元素可读取
        // 里面是通过position < limit判断是否有剩余的元素
        while (buffer.hasRemaining()) {
            System.out.println(buffer.get());
        }
        // 这时将position的位置设置成1，limit的位置设置成4
        buffer.position(1);
        buffer.limit(4);
        // 因为不能读取超过limit的元素，并且从position位置开始读取，所以这里将会输出2、4、6
        while (buffer.hasRemaining()) {
            System.out.println(buffer.get());
        }
    }
}

```
### Channel（通道）

NIO的通道类似于流，但两者之间有所区别：  
通道可以同时进行读写，而流只能读或者只能写  
通道可以实现异步读写数据  
通道可以从缓冲区读取数据，也可以写数据到缓冲区  
BIO的stream是单向的，例如FileInputStream对象只能进行读取数据的操作，而NIO中的通道（Channel）是双向的，可以读操作，也可以写操作。  
Channel在NIO中是一个接口。  
常用的Channel类有：FileChannel、DatagramChannel、ServerSocketChannel、SocketChannel。FileChannel用于文件的数据读写，DatagramChannel用于UDP的数据读写，ServerSocketChannel和SocketChannel用于TCP的数据读写。  
```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class Channel {
    public static void main(String[] args) throws Exception {
        // 从桌面上随机取一张图片进行复制操作
        // 获取原图片和被复制图片路径的流
        FileInputStream fileInputStream = new FileInputStream("/src/main/resources/img.png");
        FileOutputStream fileOutputStream = new FileOutputStream("/src/main/resources/img_1.png");
        // 通过流的getChannel()方法获取两个通道
        FileChannel fileInputStreamChannel = fileInputStream.getChannel();
        FileChannel fileOutputStreamChannel = fileOutputStream.getChannel();
        // 创建一个字节类型的缓冲区，并为其分配1024长度
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        // 每次读取图片的字节到缓冲区，当读返回-1时，表示读完了
        while (fileInputStreamChannel.read(byteBuffer) > -1) {
            // 调用flip()方法，从读的状态变为写的状态
            byteBuffer.flip();
            // 复制，将缓冲区中的数据写入到管道中
            fileOutputStreamChannel.write(byteBuffer);
            // 将缓冲区清空，以便于下一次读取
            byteBuffer.clear();
        }
        // 关闭Closeable对象
        fileOutputStreamChannel.close();
        fileInputStreamChannel.close();
        fileOutputStream.close();
        fileInputStream.close();
    }
}

```
### Selector（选择器）  
#### 基本介绍  

Java的NIO，用非阻塞的IO方式。可以用一个线程，处理多个的客户端连接，就会使用到Selector（选择器）。  
Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取时间然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。
只有在连接通道真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每一个连接都创建一个线程，不用去维护多个线程。避免了多个线程之间的上下文切换导致的开销
SelectionKey为Selector中，有一个Channel注册了，就会生成一个SelectionKey对象，在同步非阻塞中，Selector可以通过SelectionKey找到相应的Channel并处理。
SelectionKey在Selector和Channel的注册关系中一共分为四种：  

Int OP_ACCEPT：有新的网络连接可以accept，值为16（1<<4）  
int OP_CONNECT：代表连接已经建立，值为8(1<<3)  
int OP_WRITE：代表写操作，值为4(1<<2)  
int OP_READ：代表读操作，值为1(1<<0)  

#### 使用实例
客户端：  
```
package Scoket.client;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class NioClient {
    public static void main(String[] args) throws IOException {
        // 连接服务器
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("localhost", 6443));

        // 发送数据
        String message = "Hello, Server!";
        ByteBuffer buffer = ByteBuffer.wrap(message.getBytes("UTF-8"));
        socketChannel.write(buffer);
        System.out.println("Sent to server: " + message);

        // 关闭连接
        socketChannel.close();
    }
}

```
服务端  
```
package Scoket.client;  

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NioServer {
    public static void main(String[] args) throws IOException {
        // 打开 Selector
        Selector selector = Selector.open();

        // 打开 ServerSocketChannel，监听客户端连接
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.bind(new InetSocketAddress(6443));
        // 设置为非阻塞模式
        serverSocketChannel.configureBlocking(false);
        // 注册接受连接事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("Server listening on port 6443");

        while (true) {
            // 阻塞直到有就绪事件发生
            int readyChannels = selector.select();

            if (readyChannels == 0) {
                continue;
            }

            // 获取就绪事件的 SelectionKey 集合
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();

                if (key.isAcceptable()) {
                    // 有新的连接
                    handleAccept(key, selector);
                } else if (key.isReadable()) {
                    // 有数据可读
                    handleRead(key);
                }

                keyIterator.remove();
            }
        }
    }

    private static void handleAccept(SelectionKey key, Selector selector) throws IOException {
        ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
        SocketChannel socketChannel = serverSocketChannel.accept();
        socketChannel.configureBlocking(false);
        // 注册读事件
        socketChannel.register(selector, SelectionKey.OP_READ);
        System.out.println("Accepted connection from: " + socketChannel.getRemoteAddress());
    }

    private static void handleRead(SelectionKey key) throws IOException {
        SocketChannel socketChannel = (SocketChannel) key.channel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        int bytesRead = socketChannel.read(buffer);

        if (bytesRead > 0) {
            buffer.flip();
            byte[] data = new byte[bytesRead];
            buffer.get(data);
            System.out.println("Received from client: " + new String(data, "UTF-8"));

            // 在这里可以添加业务逻辑，然后将响应数据写入到 SocketChannel
            // ...

            // 关闭连接
            socketChannel.close();
        }
    }
}

```
参考源：
https://zhuanlan.zhihu.com/p/462497498
https://blog.csdn.net/a78270528/article/details/80318571
https://juejin.cn/post/6924670437867651080
https://juejin.cn/post/6925046428213608456







