# JVM钩子
## 简介
在Java应用程序中，可以通过注册关闭钩子（Shutdown Hook）函数来实现在JVM关闭时执行特定的代码。关闭钩子是一种用于在JVM关闭时执行清理任务的机制，它允许开发者在JVM关闭之前执行一些必要的清理工作，如关闭资源、保存状态等。
JVM 关闭可以分为三种  

正常关闭：当最后一个非守护线程结束或调用 System.exit() 或通过其他特定平台的特定方式关闭  
强制关闭：通过调用 Runtime.halt() 或在操作系统中直接 kill JVM进程  
异常关闭：运行时遇到 RuntimeException 异常等  

## JVM关闭时的使用场景
可以在关闭时执行一些清理临时文件，关闭资源，停止任务，停止其他服务等等操作。  

## 使用与注意事项
通过 Runtime.getRuntime().addShutdownHook() 添加到钩子列表的钩子在 JVM 关闭前会去执行这些钩子中的任务。  
即在 JVM 关闭前的最后一些收尾操作。
```
public class ShutdownHookExample {

    public static void main(String[] args) {
        // 创建并注册关闭钩子
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("Shutdown Hook is running");
            // 执行清理或关闭操作
        }));

        // 主线程业务逻辑
        System.out.println("Main thread is running");

        // 模拟应用程序执行
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // JVM关闭时，注册的关闭钩子将被执行
    }
}

```
在上述示例中，通过addShutdownHook方法注册了一个Thread实例作为关闭钩子。当JVM关闭时，该线程将执行相应的清理工作。

使用中需要注意一些使用事项  
1、如果JVM因异常关闭，那么子线程（Hook本质上也是子线程）将不会停止。但在JVM被强行关闭时，这些线程都会被强行结束  
2、关闭钩子本质上是一个线程（也称为Hook线程），用来监听JVM的关闭。通过使用 Runtime 的addShutdownHook(Thread hook) 可以向JVM注册一个关闭钩子  
3、Hook线程在JVM 正常关闭时才会执行，在强制关闭时不会执行  
4、对于一个 JVM 中注册的多个关闭钩子它们将会并发执行，所以JVM并不能保证它的执行顺序  
5、当所有 Hook线程执行完毕后，如果此时 runFinalizersOnExit（类内部定义的属性）为true，那么JVM将先运行终结器，然后停止  
6、Hook线程的执行当然会延迟 JVM 的关闭时间，这就要求在编写钩子过程中必须要尽可能减少Hook线程的执行时间  
7、由于多个钩子是一起执行的，那么很可能因为代码不当导致出现竞态条件或死锁等问题，为了避免该问题，强烈建议在一个钩子中执行一系列操作  
8、不能在钩子内调用 System.exit()，否则会卡住 JVM 的关闭过程，可以调用Runtime.halt() 替代  
9、不能在钩子中再进行钩子的添加和删除操作，否则将会抛出 IllegalStateException 异常  
10、在System.exit()之后添加的钩子无效  
11、当JVM收到 SIGTERM 命令（比如操作系统在关闭时）后，如果钩子线程在一定时间没有完成，那么Hook线程可能在执行过程中被终止  
12、Hook 线程中同样会抛出异常，如果抛出异常不处理，那么钩子的执行序列就会被停止  


引用：https://blog.csdn.net/weixin_44131922/article/details/126466336
