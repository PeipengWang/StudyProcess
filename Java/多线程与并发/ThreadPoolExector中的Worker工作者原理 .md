# ThreadPoolExector中的Worker工作者原理 

## 1 前言

Java线程池（ThreadPool）是Java中用于管理和调度线程的一种机制。线程池通过重用线程来减少创建和销毁线程的开销，从而提高应用程序的性能。Java线程池的核心组件之一是Worker线程，它负责实际的任务执行。

Worker是线程池内部的工作者，每个Worker内部持有一个线程，addWorker方法创建了一个Worker工作者，并且放入HashSet的容器中，那么这节我们就来看看Worker是如何工作的。

## 2 内部属性

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    // Worker内部持有的工作线程，就是依靠这个线程来不断运行线程池中的一个个task的
    final Thread thread;
    // 提交给这个Worker的首个任务
    Runnable firstTask;
    // 一个统计变量，记录着这个worker总共完成了多少个任务
    volatile long completedTasks;

    // 构造方法，创建Worker的时候给这个worker传入第一个运行的任务
    Worker(Runnable firstTask) {
        // 初始化AQS内部的state资源变量为-1
        setState(-1);
        // 保存一下首个任务
        this.firstTask = firstTask;
        // 使用线程工厂创建出来一个线程，这个线程负责运行任务
        this.thread = getThreadFactory().newThread(this);
    }

    // 内部的run方法，这个方法执行一个个任务
    public void run() {
        // runWorker方法，去运行一个个task
        runWorker(this);
    }

    // 实现AQS的互斥锁，这里是否持有互斥锁，不等于0就是持有
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }

    // 实现AQS加互斥锁逻辑，就是CAS将state从0设置为1，成功就获取锁
    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
    // 是新AQS释放互斥锁逻辑，就是将state变量从1设置为0，成功就释放锁成功
    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }
    // 加锁
    public void lock()        { acquire(1); }
    // 尝试加锁
    public boolean tryLock()  { return tryAcquire(1); }
    // 解锁
    public void unlock()      { release(1); }
    // 是否持有锁
    public boolean isLocked() { return isHeldExclusively(); }
}
```

可以发现：
（1）Worker工作者继承了AQS，是一个同步器
（2）Worker实现了Runnable接口
为什么要这么做呢？我们往下看。

## 3  runWorker方法

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    // 获取worker的第一个任务firstTask，赋值给task（要运行的任务）
    Runnable task = w.firstTask;
    // firstTask传给task之后，设置firstTask为null （方便firstTask完成之后垃圾回收）
    w.firstTask = null;
    // 初始化一下，将w的同步器设置为解锁状态
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 这里是重点
        // 假如task != null ，是因为上边将firstTask设置给了task，所以优先运行第一个任务firstTask
        // 假如task == null，那么调用getTask方法，从线程池的阻塞队列里面取任务出来
        while (task != null || (task = getTask()) != null) {
            // 运行任务之前需要进行加锁
            w.lock();
            // 这里就是校验一下线程池的状态
            // 如果是STOP、TIDYING、TERMINATED 需要中断一下当前线程wt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                // 这里是一个钩子，执行任务前可以做一些自定义操作
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 这里就是运行任务了，调用task的run方法执行task任务
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    // 这里就是一个后置钩子，运行完毕任务之后可以做一些自定义操作
                    afterExecute(task, thrown);
                }
            } finally {
                // 运行完task之后，需要设置task为null，否则就会死循环不断运行
                task = null;
                // 将worker完成的任务数+1,
                w.completedTasks++;
                // 解锁
                w.unlock();
            }
        }
        
        // 如果走到这里，说明跳出了上面的while循环，当前worker需要进行销毁了
        completedAbruptly = false;
    } finally {
       // 销毁当前worker
        processWorkerExit(w, completedAbruptly);
    }
}
```

![img](https://img2023.cnblogs.com/blog/990230/202304/990230-20230411210806341-2112766235.png)

上面的核心流程主要是：
（1）在一个while循环里面，不断的运行任务task，task 的来源可能有两种
（2）task可能是创建worker的时候传入的firstTask，或者是调用getTask方法从阻塞队列取出的task
（3）每次调用task.run方法执行任务之前，需要先加锁，然后运行task任务，然后释放锁锁
（4）每次循环前，如果获取运行的task任务为null，则需要跳出while循环，准备销毁这个worker了

## 4 getTask方法

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?
    // 在一个循环里面，进行重试
    for (;;) {
        // 获取当前线程池的控制变量（包含线程池状态、线程数量）
        int c = ctl.get();
        // 获取当前线程池的状态
        int rs = runStateOf(c);

        // 如果当前线程池状态是STOP、TIDYING、TERMINATED，则说明线程池关闭了，直接返回null
        // 如果当前线程池状态为SHUTDOWN、并且阻塞队列是空，说明线程池即将关闭，并且没有多余要执行的任务了，直接返回ull
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        // 计算当前线程池中线程数量wc
        int wc = workerCountOf(c);

        // 注意这里的timed控制变量很重要，表示从阻塞队列中获取任务的时候，是一直阻塞还是具有超时时间的阻塞
        // （1）当前线程数量 wc > corePoolSize 的时候为true
        // 假如corePoolSize = 5, maximumPoolSize = 0, wc = 8
        // 当前线程数8 > corePoolSize，那么多出的这3个线程，在keepAliveTime时间内空闲就干掉
        // (2) 当allowCoreThreadTimeout 为true，则timed为true
        // 这里的意思是，线程池内的线程，只要超过keepAliveTime空闲的全部干掉
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // wc > maximumPoolSize 正常情况不可能发生的
        // 这里的意思是如果超时了，超时了还取不到任务，就返回null
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // 这的意思就是
            // 如果timed == true， 从阻塞队列取任务最多阻塞keepAliveTime时间，如果娶不到返回null
            // 如果timed == false。则调用take方法从阻塞队列取任务，一直阻塞，知道取到任务位置
            // 这里涉及的一些阻塞队列的知识，我们在上一篇并发容器的时候已经非常深入的分析过了
            Runnable r = timed ?
                // 调用poll方法，最多阻塞keepAliveTime时间
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                // 调用take方法，在取到任务之前会一直阻塞
                workQueue.take();
            // 如果从队列取到任务，直接返回
            if (r != null)
                return r;
            // 否则就是超时了
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

![img](https://img2023.cnblogs.com/blog/990230/202304/990230-20230411211022551-1118780214.png)

上面的核心流程主要是：
（1）判断一下当前线程池的状态，如果是STOP、TIDYING、TERMINATED状态中的一种，那么直接返回null，别执行任务了，线程池就要销毁了，赶紧销毁掉所有的worker
（2）如果是SHUTDOWN，并且workerQueue阻塞队列是空，说明线程池即将关闭，并且没有多余的任务了，所以worker也可以被销毁了
（3）如果当前线程数量 wc > corePoolSize，也就是线程数量大于核心线程数，此时多出的这部分线程在keepAliveTimeout时间内没能从阻塞队列取出任务，则返回null，也要销毁这些多出的worker
（4）如果allowCoreThreadTimeout == true，这个表示允许销毁所有线程包括核心线程，就是任意一个线程超过keepAliveTimeout时间内没取到任务，就会被干掉。

## 5 空闲线程被销毁

最后看下Worker是怎么被销毁的：

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    // completeAbruptly表示当前线程是否因为被中断而被销毁的
    // 如果是正常情况，因为keepAliveTimeout空闲而被销毁，则为false
    if (completedAbruptly)
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    // 加锁
    mainLock.lock();
    try {
        // 计算一下线程池完成总的任务数量
        completedTaskCount += w.completedTasks;
        // 从HashSet容器中移除当前的worker
        workers.remove(w);
    } finally {
        // 释放锁
        mainLock.unlock();
    }
    
    // 尝试中止线程池，这里我们后面的章节再分析
    tryTerminate();

    int c = ctl.get();
    // 如果当前线程状态为RUNNING、SHUTDOWN
    if (runStateLessThan(c, STOP)) {
        if (!completedAbruptly) {
            // 这就是计算一下当前线程池允许的最小线程数
            // 正常情况是min=corePoolSize，但是当allowCoreThreadTimeout为true时候，允许销毁所有线程，则min=0
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            // 如果min = 0 并且 阻塞队列非空，说明还有任务没执行
            // 此时最少要保留1个线程去运行这些任务，不能销毁所有
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
             // 如果当前线程数量 >= min值，可以了，销毁动作结束了
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        // 说明completedAbruptly == true，说明可能是因为线程被中断（interrupted方法）而被销毁
        // 此时可能还有很多任务还没执行，需要加会容器里面
        addWorker(null, false);
    }
}
```

上面主要就是做了：
（1）如果是正常情况keepAliveTimeout空闲时间被销毁，则从HashSet容器里面移除即可
（2）如果当前线程池状态是RUNNING、SHUTDOWN，说明还有一些任务没执行。
从HashSet容器移除当前worker之后，需要判断一下如果worker是因为异常情况被中断，需要新创建worker来继续执行任务
我们这里看到worker继承了AQS，每次worker执行任务之前都需要lock加锁，这是为什么呢？那就需要看下interruptIdleWorker方法了，interruptIdleWorker方法是销毁空闲的线程的意思。

```java
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 遍历容器中的所有worker
        for (Worker w : workers) {
            Thread t = w.thread;
            // 这里核心点就是要调用w.tryLock方法尝试获取锁
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    // 只有获取w工作者的互斥锁之后才能中断它
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    // 释放锁
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```

我们可以发现：
（1）因为Worker执行每个task的之前，都要执行w.lock，对worker进行加锁，然后才能执行task任务。
（2）此时如果有别的线程要中断Worker，也需要获取w的互斥锁，执行w.tryLock()方法。
（3）如果执行tryLock加锁失败，说明当前Worker方法正在执行task，不允许中断，否则可能task执行一半，线程就被中断，导致一些数据异常问题。
所以说加锁就是为了防止Worker正在执行任务的时候，被人直接中断，导致异常。

其实ThreadPoolExecutor方法提供了**直接暴力中断所有线程的方法**，也就是不管当前Worker是否正在执行task任务，都会直接被中止掉的。而interruptIdleWorkers()这个方法是比较优雅的进行中断，中断worker之前，会先获取锁，如果失败则不允许中断。关于暴力中止线程池、以及其它中止线程池的方式，我们后面会说。

