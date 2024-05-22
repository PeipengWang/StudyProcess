# ReetrantLock

## 1、基本使用

1、定义锁住的代码

```
package lock.ReetrantTest;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class MyService {
    Lock lock = new ReentrantLock();
    public void testMethod(){
        lock.lock();
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread() + ":" + i);
        }
        lock.unlock();
    }
}

```

2、定义线程

```
package lock.ReetrantTest;

public class MyThread extends Thread{
    private MyService service;

    public MyThread(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
       service.testMethod();
    }
}

```

3、执行测试代码

```
package lock.ReetrantTest;

public class Test01 {
    public static void main(String[] args) {
        MyService service = new MyService();
        new MyThread(service).start();
        new MyThread(service).start();
        new MyThread(service).start();
        new MyThread(service).start();
    }

}
```

输出为：

```
Thread[Thread-0,5,main]:0
Thread[Thread-0,5,main]:1
Thread[Thread-0,5,main]:2
Thread[Thread-0,5,main]:3
Thread[Thread-0,5,main]:4
Thread[Thread-2,5,main]:0
Thread[Thread-2,5,main]:1
Thread[Thread-2,5,main]:2
Thread[Thread-2,5,main]:3
Thread[Thread-2,5,main]:4
Thread[Thread-1,5,main]:0
Thread[Thread-1,5,main]:1
Thread[Thread-1,5,main]:2
Thread[Thread-1,5,main]:3
Thread[Thread-1,5,main]:4
Thread[Thread-3,5,main]:0
Thread[Thread-3,5,main]:1
Thread[Thread-3,5,main]:2
Thread[Thread-3,5,main]:3
Thread[Thread-3,5,main]:4

Process finished with exit code 0

```

可以看到，数据都是顺序输出的，但是线程是哪个先开启并不一定先执行，这个是非公平锁

可以改成公平锁

```
package lock.ReetrantTest;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class MyService {
    Lock lock = new ReentrantLock(true);
    public void testMethod(){
        lock.lock();
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread() + ":" + i);
        }
        lock.unlock();
    }
}
```

## 2、ReentrantLock  +  Condition实现选择性通知

通过Condition进行选择性通知

首先编写逻辑代码，里面又饿await和single

```
package lock.ReetrantTest;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class MyService {
    private ReentrantLock lock = new ReentrantLock(true);
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();


    public void await(){

        try {
            lock.lock();
            System.out.println("awit Time start : " + System.currentTimeMillis());
            condition1.await();
            System.out.println("awit Time end : " + System.currentTimeMillis());
        } catch (InterruptedException e) {
           e.printStackTrace();
        }finally {
            System.out.println("release lock");
            lock.unlock();;
        }
    }

    public void signal(){
        try {
            lock.lock();
            System.out.println("single:"+System.currentTimeMillis());
            condition1.signal();
        }finally {
            lock.unlock();
        }

    }
}

```

线程代码

```
package lock.ReetrantTest;

public class MyThread extends Thread{
    private MyService service;

    public MyThread(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
       service.await();
    }
}

```

测试方法

```
package lock.ReetrantTest;

public class Test01 {
    public static void main(String[] args) {
        MyService service = new MyService();
        new MyThread(service).start();
        new MyThread(service).start();
        new MyThread(service).start();
        new MyThread(service).start();
        service.signal();
        service.signal();
        service.signal();
    }
}

```

输出

```
awit Time start : 1716344465163
awit Time start : 1716344465164
awit Time start : 1716344465165
single:1716344465165
awit Time end : 1716344465165
release lock
awit Time start : 1716344465166
single:1716344465166
awit Time end : 1716344465166
release lock
single:1716344465166
awit Time end : 1716344465166
release lock
```

可以看到，single执行了三次，线程release lock就三个，但是awit Time start有四个

**await让线程暂停的原理是什么呢？**

并发包内部执行了Unsafe类中的public native void park(boolean isAbsolute, long time)方法，让线程呈暂停状态，方法参数isAbsolute代表是否为绝对时间，方法参数time代表时间值。

测试代码：

```
package lock.ReetrantTest;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

public class Test02 {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);
        long start = System.currentTimeMillis();
        System.out.println("begin " + System.currentTimeMillis());
        System.currentTimeMillis();
        unsafe.park(true, System.currentTimeMillis() + 3000);
        System.out.println("end " + System.currentTimeMillis());
        System.out.println(System.currentTimeMillis() - start);
    }
}
```

程序运行结果如下

```
begin 1716346263026
end 1716346266026
3000
```

## 3、实现生产者与消费者的交替输出

服务类

```

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class MyService {
    private ReentrantLock lock = new ReentrantLock(true);
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private boolean hasValue = false;

    public void set(){

        try {
            lock.lock();
            if(hasValue) {
                condition1.await();
            }
            hasValue = true;
            System.out.println("1");
            condition1.signal();
        } catch (InterruptedException e) {
           e.printStackTrace();
        }finally {
            lock.unlock();;
        }
    }
    public void get(){

        try {
            lock.lock();
            if(!hasValue) {
                condition1.await();
            }
            hasValue = false;
            System.out.println("2");
            condition1.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();;
        }
    }
}
```

实现两个线程分别执行这个服务的两个方法

```



public class MyThreadA extends Thread{
    private MyService service;

    public MyThreadA(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            service.get();
        }

    }
}


public class MyThreadB extends Thread{
    private MyService service;

    public MyThreadB(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            service.set();
        }

    }
}

```

测试方法

```
package lock.ReetrantTest.Provider;

public class Test03 {
    public static void main(String[] args) {
        final MyService service = new MyService();
        new MyThreadA(service).start();
        new MyThreadB(service).start();
    }
}

```

可以看到生产者消费者交替输出

```
1
2
1
2
1
2
1
2
1
2
1
2
1
2
1
2
1
2
1
2

Process finished with exit code 0
```

## 4、getHoldCount用法

public int getHoldCount() 方法作用是查询“当前线程”保持次锁定的个数，即调用lock()的次数。注意是当前线程，实际的意思是重入锁的次数.

执行lock()会使得可重入锁次数加1，执行unlock()会使可重入锁次数减1。



测试代码

```
    public void testMethodA(){
        lock.lock();
        System.out.println("getHoldCount:" + lock.getHoldCount());
        testMethodB();
        System.out.println("getHoldCount:" + lock.getHoldCount());
        lock.unlock();
        System.out.println("getHoldCount:" + lock.getHoldCount());
    }
    private void testMethodB(){
        try {
            lock.lock();
            System.out.println("getHoldCount:" + lock.getHoldCount());
        }finally {
            lock.unlock();
        }
    }
```

调用

```
        MyService service = new MyService();
        new MyThread(service).start();
        System.out.println(service.getHoldCount());
```

输出

```
getHoldCount:1
getHoldCount:2
getHoldCount:1
getHoldCount:0
```

## 5、getQueueLength():获取正在等待获取此锁的线程估计数

```
    public void testMethodC(){
        try {
            lock.lock();
            Thread.sleep(1000);
            System.out.println("getQueueLength:" + lock.getQueueLength());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

    }
```

开启四个线程测试

```
        MyService service = new MyService();
        new MyThread(service).start();
        new MyThread(service).start();
        new MyThread(service).start();
        new MyThread(service).start();
```

会发现

```
getQueueLength:3
getQueueLength:2
getQueueLength:1
getQueueLength:0
```

开始会有3个线程在等待

## 6、其他方法

getWaitQueueLength(Condition condition):返回给定condition的线程计数，实际上是查看某个condition的await个数

hasQueueThread(Thread thread)：查询指定的线程是否正在等待获取此锁，也就是判断参数中的线程是否在等待队列中。

hasQueuedThreads()：查看是否有线程获取等待此锁

hasWaiters(Condition condition)：查询是否有线程正在等待此锁有关的condition条件

isHeldByCurrentThread()：查询当前线程是否保持此锁

isLocked():次锁是否由任意线程保持

tryLock()：嗅探拿锁

lockInterruptibly()：当某个线程尝试获得锁并且堵塞在lockInterruptibly()方法时，该线程可以被中断。

## 7、实现线程的顺序执行

类似于3，只需要将 private boolean hasValue = false;变成一个实际的值，线程跟这个值对应起来即可。

## 8、读写锁

ReetrantLock是一个互斥锁，但是有些情况下两个线程都是读可以不用互斥，于是有了ReetrantReadWriteLock。

基本情况就是

读写互斥

写写互斥

读读共享

定义方法

```
        ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
        lock.readLock().lock();//加读锁
        lock.writeLock().lock();//加写锁
        lock.writeLock().unlock();//解锁
        lock.readLock().unlock();
```


