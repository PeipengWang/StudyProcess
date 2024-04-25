# Java进程的两种启动方式
Runtime使用方式
## Runtime使用方式
通过Runtime实例，使得应用程序和其运行环境相连接。Runtime是在应用启动期间自动建立，应用程序不能够创建Runtime，但是我们可以通过Runtime.getRuntime()来获得当前应用的Runtime对象引用，通过该引用我们可以获得当前运行环境的相关信息，比如空闲内存、最大内存以及为当前虚拟机添加关闭钩子（addShutdownHook（）），执行指定命令（exec（））等。

每个 Java 应用程序都有一个类实例 Runtime，允许应用程序与应用程序运行的环境进行交互。当前运行时间可以从getRuntime方法中获取。
应用程序无法创建自己的此类实例。

### addShutdownHook（）
#### jvm关闭介绍
jvm的关闭方式有三种：
正常关闭：当最后一个非守护线程结束或者调用了System.exit或者通过其他特定平台的方法关闭（发送SIGINT，SIGTERM信号等）
强制关闭：通过调用Runtime.halt方法或者是在操作系统中直接kill(发送SIGKILL信号)掉JVM进程
异常关闭：运行中遇到RuntimeException异常等。
#### 关闭钩子介绍
在某些情况下，我们需要在JVM关闭时做些扫尾的工作，比如删除临时文件、停止日志服务以及内存数据写到磁盘等，为此JVM提供了关闭钩子（shutdown hooks）来做这些事情。另外特别注意的是：如果JVM因异常关闭，那么子线程（Hook本质上也是子线程）将不会停止。但在JVM被强行关闭时，这些线程都会被强行结束。
关闭钩子本质是一个线程（也称为Hook线程），用来监听jvm的关闭。通过Runtime的addShutdownHook可以向JVM注册一个关闭钩子。Hook线程在JVM正常关闭才会执行，强制关闭时不会执行。
JVM中注册的多个关闭钩子是并发执行的，无法保证执行顺序，当所有Hook线程执行完毕，runFinalizersOnExit为true,JVM会先运行终结器，然后停止。
注意事项：
1.hook线程会延迟JVM的关闭时间，所以尽可能减少执行时间。
2.关闭钩子中不要调用system.exit()，会卡主JVM的关闭过程。但是可以调用Runtime.halt()
3.不能再钩子中进行钩子的添加和删除，会抛IllegalStateException
4.在system.exit()后添加的钩子无效，因为此时jvm已经关闭了。
5.当JVM收到SIGTERM命令（比如操作系统在关闭时）后，如果钩子线程在一定时间没有完成，那么Hook线程可能在执行过程中被终止。
6.Hook线程也会抛错，若未捕获，则钩子的执行序列会被停止。
优雅的关闭线程
```
    private static ThreadPoolExecutor THREAD_POOL;

    private static final int MAX = 8;

    static {
        int max = Runtime.getRuntime().availableProcessors();
        BlockingQueue<Runnable> queue = new ArrayBlockingQueue<>(200);
        RejectedExecutionHandler rejectedExecutionHandler = new ThreadPoolExecutor.CallerRunsPolicy();

        THREAD_POOL = new ThreadPoolExecutor(1, max > MAX ? MAX : max, 60L, TimeUnit.SECONDS, queue, rejectedExecutionHandler);
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            // 使新任务无法提交.
            THREAD_POOL.shutdown();

            try {
                // 等待5秒如果还没有关闭，则强制关闭
                if (THREAD_POOL.awaitTermination(5L, TimeUnit.SECONDS)) {
                    THREAD_POOL.shutdownNow();
                }
            } catch (InterruptedException e) {
                log.error("线程关闭异常", e);
                Thread.currentThread().interrupt();
                THREAD_POOL.shutdownNow();
            }
        }));
    }
```
### exec（）
Runtime.getRuntime().exec 用于调用外部可执行程序或系统命令，并重定向外部程序的标准输入、标准输出和标准错误到缓冲池。功能和windows“运行”类似。
格式：
```
Process process = Runtime.getRuntime().exec( ".//p.exe ");
process.waitfor();
```
第一行的“.//p.exe”是要执行的程序名，Runtime.getRuntime() 返回当前应用程序的Runtime对象，该对象的 exec() 方法指示Java虚拟机创建一个子进程执行指定的可执行程序，并返回与该子进程对应的Process对象实例。通过Process可以控制该子进程的执行或获取该子进程的信息。
第二条语句的目的等待子进程完成再往下执行。
方法API
```
// 在单独的进程中执行指定的外部可执行程序的启动路径或字符串命令
public Process exec(String command)
// 在单独的进程中执行指定命令和变量
public Process exec(String[] cmdArray)
// 在指定环境的独立进程中执行指定命令和变量
public Process exec(String command, String[] envp)
// 在指定环境的独立进程中执行指定的命令和变量
public Process exec(String[] cmdArray, String[] envp)
// 在指定环境和工作目录的独立进程中执行指定的字符串命令
public Process exec(String command, String[] envp, File dir)
// 在指定环境和工作目录的独立进程中执行指定的命令和变量
public Process exec(String[] cmdarray, String[] envp, File dir)
// 参数说明：
cmdarray // 包含所调用命令及其参数的数组。数组第一个元素是命令，其余是参数
envp // 字符串数组，其中每个元素的环境变量的设置格式为 name=value，如果子进程应该继承当前进程的环境，则该参数为null
dir // 子进程的工作目录；如果子进程应该继承当前进程的工作目录，则该参数为null
// 参数cmdArray 示例：shutdown -s -t 3600
String arr[] = {"shutdown","-s","-t","3600"};
Process process = Runtime.getRuntime().exec(arr[]);
/*
注意：
在调用这个方法时，不能将命令和参数放在一起，eg：String arr[] = {"shutdown -s -t 3600"};
这样会导致程序把“shutdown -s -t 3600”当成是一条命令的名称，然后去查找“shutdown -s -t 3600”这条命令，它当然会找不到，所以就会报错
*/
```
Processc常用方法
```
// 导致当前线程等待，如有必要，一直要等到由该 Process 对象表示的进程已经终止。
int waitFor()
/* 如果已终止该子进程，此方法立即返回。
如果没有终止该子进程，调用的线程将被阻塞，直到退出子进程，0 表示正常终止 */
// 杀掉子进程
void destroy()
// 返回子进程的出口值，值 0 表示正常终止
int exitValue()
// 获取子进程的错误流
InputStream getErrorStream()
// 获取子进程的输入流
InputStream getInputStream()
// 获取子进程的输出流
OutputStream getOutputStream()

```
exec的陷阱https://www.cnblogs.com/fpqi/p/9679039.html
遵循这些法则,以避免的陷阱在运行时执行():
你不能从外部过程获得一个退出状态，直到它已经退出
你必须从你外部程序立即处理输入、输出和错误流
您必须使用运行时exec()来执行程序
你不能使用运行时执行()就像一个命令行

## ProcessBuilder使用方式
ProcessBuilder 用于创建操作系统进程。 其start()方法创建具有以下属性的新Process实例：
命令
环境
工作目录
输入来源
标准输出和标准错误输出的目标
redirectErrorStream

```
// win
new ProcessBuilder("exec").start()

// mac 注意，使用下面这个，则传参不能是 open -n xxx
new ProcessBuilder("/Applications/Calculator.app/Contents/MacOS/Calculator").start()
```

使用上面这种姿势，特别需要注意的是内部传参不能是open -n
