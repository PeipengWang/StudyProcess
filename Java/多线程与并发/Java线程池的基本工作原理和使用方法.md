本文主要整理了一些关于线程池的资料，首先对线程池的基本工作原理进行汇总，然后介绍在Java中线程池的核心组件，再介绍线程池的拒绝策略，最后介绍java中五种线程池的基本概念。  

## 1、基本工作原理
JVM首先根据用户参数创建一定数量的可运行的线程任务，并将其放入队列中，在创建线程后启动这些任务，如果创建的线程数目超过了最大线程数量（用户设置的线程数目），则超出数量的线程排队等候，在有任务执行完毕欧，线程池调度器会发现有可用线程，进而再次从队列中取出任务并且执行。  
线程池的主要任务是：线程复用、线程资源管理、控制操作系统的最大并发数，以保证系统高效且安全的运行。  
## 2、线程池的核心组件和核心类
四个核心组件：  
①线程池管理器：用于创建并管理线程  
②工作线程：线程池中执行具体任务的线程  
③任务接口：用于定义工作线程的调度和执行策略，只有你线程实现了接口，线程任务才能被线程调度  
④任务队列：存放待处理的任务，新的任务会不断被加入队列中，完成执行的任务将被从队列中移除  
ThreadPoolExecutor是构建线程的核心方法  
```
 public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
        this.prestartAllCoreThreads();
    }
```
具体构造参数如表所示：  

参数     | 说明
-------- | -----
corePoolSize | 线程池中核心线程的数量
maximumPoolSize| 线程池中最大线程的数量
keepAliveTime | 当线程数量超过corePoolSize时，空闲线程存活时间
unit | keepALiveTime的时间单位      |
workQueue| 任务队列，被提交但尚未被执行的任务存放的地方|
threadFactory |线程工厂，用于创建线程，可使用默认的线程工厂或者自定义的线程工厂 |
handler| 由于任务过多或者其他原因导致线程池无法处理时的拒绝策略|
## 3、Java线程池的工作流程
Java线程池的工作流程为：线程刚刚被创建的时候，只是向任务系统申请一个用于执行线程队列和管理线程池的线程资源。在executor（）添加一个任务时，线程池会按照以下流程执行任务：  
①正在运行的线程池数量少于corePoolSize，线程池会立即创建线程并且执行任务。  
②如果正在运行的线程数量大于corePoolSize，该任务会被放入阻塞队列中。  
③在阻塞队列中已满且正在运行的线程数量少于maximumPoolSize时，线程池会创建非核心线程立刻执行该线程任务。  
④在阻塞队列已满并且正在运行的线程数量大于等于maxmumPoolSize时，线程池将会拒绝执行该线程任务并抛出RejectExecutionException异常。  
⑤在线程任务执行完毕后，该任务将会从线程池中移除，线程池将从队列中取下一个任务继续执行。  
⑥在线程处于空闲状态的线程时间超过keepAliveTime时间时，正在运行的线程数量超过corePoolSize，该线程将会被认定为空闲线程并且停止。  
## 4、线程池的拒绝策略
如果线程池中的核心线程被用完且阻塞队列已排满，且此线程池的资源已经耗尽，线程池没有足够的资源执行新的任务。为了保证操作系统的安全，线程池将会通过拒绝策略处理添加新的任务。JDK总共有四种拒绝策略，如下所示：  
AbortPolice：直接抛出异常，组织线程正常运行  
CallerRunsPolicy：如果未丢弃的线程任务未关闭，则执行该线程任务  
DiscardOldesPolicy：移除线程中最早的一个线程任务  
DiscardPolicy：丢弃当前线程而不做任何处理  
补充：还有自定义拒绝策略。  
## 5、五种线程池
①可缓存的线程池（newCachedThreadPool）  
定义方法：`ExecutorService catchedThreadPool  = Executors().newCachedThreadPool();`  
他在创建新的线程的时候，如果有可重用的线程，则重用他们，否则就创建一个新的线程并将其添加到线程池中。很大程度上因为重用线程池提高性能。  
在线程池的keepAliveTime时间超过60s后，该线程会被终止并且移除，因此在没有线程运行的时候，newCarchedThreadPool不会占用线程资源   
②固定大小的线程池（newFixedThreadPool）  
创建一个固定线程数量的线程池，并且将线程资源都放在队列中循环使用。  

```
ExecutorService fixedThreadPool  = Executors.newFIxedThreadPool(5);
```
③可做任务调度的线程池（newSheduledThreadPool）  
可以设置给定延迟时间后执行，或者定期执行某个线程任务  
④单个线程池的线程（newSingleThreadExecutor）  
保证线程池中运行的只有一个线程，当该线程停止或者发生异常时，会启动一个新的线程来代替这个线程。  
⑤足够大小的线程池（newWorkStealingPool）  
创建足够线程的线程池来达到快速运算的目的，在内部使用多个队列来减少各个线程调度产生的竞争。  
