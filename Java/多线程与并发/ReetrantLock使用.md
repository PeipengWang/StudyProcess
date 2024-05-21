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

