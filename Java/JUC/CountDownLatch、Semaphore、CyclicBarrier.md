## CountDownLatch

### 作用

CountDownLatch（倒计时门闩）是Java并发包中的一个工具类，它可以让一个或多个线程等待其他线程执行完毕后再继续执行。它的作用是在某些并发任务需要等待其他任务执行完毕后才能执行的场景下实现线程间的同步。  

它的主要特点是：  

可以让一个或多个线程等待其他线程执行完毕。  
允许在任意时刻将计数减少，但一旦计数减至零，就会释放所有等待的线程。  
一个常见的实例是，在主线程中等待多个子线程全部执行完毕后再执行某些操作。  


### join和CountLatch的区别
join 和 CountDownLatch 都是 Java 中用于线程间协作的工具，但它们在实现上有一些不同之处，主要体现在以下几个方面：

**功能：**  
join：join 是 Thread 类的方法，用于让当前线程等待调用 join 方法的线程执行完毕后再继续执行。
CountDownLatch：CountDownLatch 是一个并发工具类，用于实现一个或多个线程等待其他线程执行完毕后再继续执行。

**使用方式：** 
join：在一个线程对象上调用 join 方法，该线程会等待调用 join 方法的线程执行完毕后再继续执行。    
CountDownLatch：创建一个 CountDownLatch 实例，指定初始计数值，然后在一个或多个线程中调用 countDown 方法来减小计数值，其他线程可以通过 await 方法来等待计数值变为零。    

**计数控制：**

join：join 方法只能等待一个线程执行完毕，不能灵活地控制等待多个线程。  
CountDownLatch：CountDownLatch 可以灵活地控制等待的线程数量，只需指定初始计数值，并在适当的时候调用 countDown 方法即可。  

**复用性：**  
join：join 方法一旦被调用，就会使得当前线程阻塞等待被调用的线程执行完毕，不能被重复使用。  
CountDownLatch：CountDownLatch 可以在计数值变为零后重复使用，只需重新设置初始计数值即可。  

**适用场景：**  
join 适用于在主线程中等待某个特定线程执行完毕后再执行后续操作的场景。  
CountDownLatch 适用于实现多个线程之间的协作，例如一个线程等待其他多个线程执行完毕后再继续执行的场景。  

### 实例

#### **实例join方式**  

```
public class TestJoin {
   public static void main(String[] args) throws InterruptedException {
      System.out.println(Thread.currentThread().getName()+" start");
      ThreadTest t1=new ThreadTest("A");
      ThreadTest t2=new ThreadTest("B");
      ThreadTest t3=new ThreadTest("C");
      System.out.println("t1start");
      t1.start();
      System.out.println("t1end");
      System.out.println("t2start");
      t2.start();
      System.out.println("t2end");
      //在join之前，都是当前线程在执行。
      t1.join();
      System.out.println("t3start");
      t3.start();
      System.out.println("t3end");
      System.out.println(Thread.currentThread().getName()+" end");
   } 
}
```

结论： t.join()方法只会使主线程或调用t.join()的线程进入等待池，等待 t 线程执行完毕后才会被唤醒，但并不影响同一时刻处在运行状态的其他线程。

join()方法的底层是利用wait()方法实现。  
join()方法是一个同步方法，当主线程调用t1.join()方法时，主线程先获得了t1对象的锁。  
join()方法中调用了t1对象的wait()方法，使主线程进入了t1对象的等待池。  

#### CountDownLatch方式  

也是join的方式  
当调用latch.wait()时，只有latch.getCount()==0才会唤醒   
实例  

```
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(2);
        Thread thread1 = new Thread(() -> {
            System.out.println(latch.getCount());
            System.out.println("Thread 1 is running");
            latch.countDown();
        });

        Thread thread2 = new Thread(() -> {
            System.out.println(latch.getCount());
            System.out.println("Thread 2 is running");

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("Thread 2 is end");
            latch.countDown();
        });

        Thread thread3 = new Thread(() -> {
            System.out.println(latch.getCount());
            System.out.println("Thread 3 is running");
            try {
                latch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(latch.getCount());
            System.out.println("Thread 3 is end");
        });

        thread1.start();
        thread2.start();
        thread3.start();

        latch.await(); // Main thread waits until count down to zero

        System.out.println("All threads have finished");
    }
}

```





## CyclicBarrier

### 作用

`CyclicBarrier` 是 Java 并发包中的一个同步辅助类，它允许一组线程互相等待，直到所有线程都到达某个屏障点（barrier），然后再一起继续执行。它的作用和场景通常是在一组线程需要相互等待并且在到达某个状态后一起执行下一步操作的情况下使用。

`CyclicBarrier` 的主要特点包括：

1. **协调多个线程**：`CyclicBarrier` 可以协调多个线程，让它们在到达某个屏障点后进行等待，直到所有线程都到达，然后再一起继续执行后续操作。
2. **循环使用**：`CyclicBarrier` 的屏障点可以被循环使用。一旦所有线程都到达屏障点后，它就会被重置，可以被再次使用。
3. **同步点**：`CyclicBarrier` 的等待点是同步点，线程到达同步点后会等待其他线程到达，直到所有线程都到达后才会一起继续执行。
4. **可设置回调函数**：可以在所有线程到达屏障点后，最后一个到达的线程执行一个特定的动作（通过构造函数中的 `Runnable` 参数）。

### `CyclicBarrier` 的原理

- `CyclicBarrier` 内部维护了一个计数器，用于记录还需要等待的线程数量。
- 每个线程到达屏障点时，都会调用 `await()` 方法，该方法会将当前线程阻塞，并等待其他线程到达。
- 每当一个线程到达屏障点，计数器就会减一，直到计数器为零，表示所有线程都已经到达。
- 当计数器为零时，所有线程被释放，并且屏障点被重置，可以被下一轮使用。

### CyclicBarrier`的使用：

```java
java复制代码import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
    public static void main(String[] args) {
        int parties = 3; // 指定线程数量
        Runnable barrierAction = () -> System.out.println("All parties have reached the barrier.");
        CyclicBarrier barrier = new CyclicBarrier(parties, barrierAction);

        for (int i = 0; i < parties; i++) {
            Thread thread = new Thread(new Worker(barrier));
            thread.start();
        }
    }

    static class Worker implements Runnable {
        private final CyclicBarrier barrier;

        Worker(CyclicBarrier barrier) {
            this.barrier = barrier;
        }

        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting at the barrier.");
                barrier.await();
                System.out.println(Thread.currentThread().getName() + " has passed the barrier.");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

在上述示例中，创建了一个 `CyclicBarrier` 实例，指定了线程数量为 3，并且设置了一个回调函数，在所有线程都到达屏障点后打印一条消息。然后创建了三个线程，每个线程执行到达屏障点后调用 `await()` 方法等待其他线程，当所有线程都到达后，屏障点被触发，所有线程继续执行。

## Semaphore

Semaphore主要用于管理信号量，同样在创建Semaphore对象实例的时候通过传入构造参数设定可供管理的信号量的数值。简单说，信号量管理的信号就好比令牌，构造时传入令牌数量，也就是Semaphore控制并发的数量。线程在执行并发的代码前要先获取信号（通过aquire函数获取信号许可），执行后归还信号（通过release方法归还），每次acquire信号成功后，Semaphore可用的信号量就会减一，同样release成功之后，Semaphore可用信号量的数目会加一，如果信号量的数量减为0，acquire调用就会阻塞，直到release调用释放信号后，aquire才会获得信号返回。

`Semaphore` 是 Java 并发包中的一个同步工具类，用于控制同时访问特定资源的线程数量。它的作用是限制可以同时访问某个资源或某组资源的线程数量，从而保护共享资源不被过度使用。`Semaphore` 主要用于解决多线程并发访问共享资源时的同步控制问题。

`Semaphore` 的主要特点包括：

1. **控制并发访问**：`Semaphore` 可以限制同时访问某个资源的线程数量，从而控制并发访问的程度。
2. **可用于限流**：`Semaphore` 可以用于实现限流的效果，控制并发访问某个资源的速率。
3. **可以是公平或非公平的**：`Semaphore` 支持公平性选择，即可以选择是否按照线程的到达顺序来获取许可。

`Semaphore` 类有两个主要的方法：

- `acquire()`：获取一个许可，如果没有可用的许可，则线程将阻塞，直到有许可可用。
- `release()`：释放一个许可，将其返回到信号量中。

下面是一个简单的示例，演示了 `Semaphore` 的使用：

```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
    public static void main(String[] args) {
        int permits = 2; // 设置信号量的许可数量
        Semaphore semaphore = new Semaphore(permits);

        // 创建并启动多个线程
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(new Worker(semaphore));
            thread.start();
        }
    }

    static class Worker implements Runnable {
        private final Semaphore semaphore;

        Worker(Semaphore semaphore) {
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire(); // 获取许可
                System.out.println(Thread.currentThread().getName() + " is accessing the resource.");
                Thread.sleep(2000); // 模拟访问资源
                System.out.println(Thread.currentThread().getName() + " has finished accessing the resource.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release(); // 释放许可
            }
        }
    }
}

```

在上述示例中，创建了一个 `Semaphore` 实例，初始许可数量为 2。然后创建了五个线程，每个线程在执行时先获取一个许可，然后访问资源，访问完毕后释放许可。由于许可数量为 2，因此最多只有两个线程能够同时访问资源，其余线程需要等待前面的线程释放许可后才能继续执行。







