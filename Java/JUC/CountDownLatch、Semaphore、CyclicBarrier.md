## join和CountDownLatch

join代码

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

CountDownLatch  也是join的方式
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

Semaphore
Semaphore主要用于管理信号量，同样在创建Semaphore对象实例的时候通过传入构造参数设定可供管理的信号量的数值。简单说，信号量管理的信号就好比令牌，构造时传入令牌数量，也就是Semaphore控制并发的数量。线程在执行并发的代码前要先获取信号（通过aquire函数获取信号许可），执行后归还信号（通过release方法归还），每次acquire信号成功后，Semaphore可用的信号量就会减一，同样release成功之后，Semaphore可用信号量的数目会加一，如果信号量的数量减为0，acquire调用就会阻塞，直到release调用释放信号后，aquire才会获得信号返回。

CyclicBarrier
等待几个线程都到了wait状态才开始启动，启动之前可以先执行其他线程，等这个线程执行完成后继续执行这几个wait状态的线程

并行计算，异步处理的接口：Future FutureTask

什么是CompletionService？
当我们使用ExecutorService启动多个Callable时，每个Callable返回一个Future，而当我们执行Future的get方法获取结果时，可能拿到的Future并不是第一个执行完成的Callable的Future，就会进行阻塞，从而不能获取到第一个完成的Callable结果，那么这样就造成了很严重的性能损耗问题。而CompletionService正是为了解决这个问题，它是Java8的新增接口，它的实现类是ExecutorCompletionService。CompletionService会根据线程池中Task的执行结果按执行完成的先后顺序排序，任务先完成的可优先获取到。

中断：
而 Thread.interrupt 的作用其实也不是中断线程，而是「通知线程应该中断了」，
具体到底中断还是继续运行，应该由被通知的线程自己处理。

具体来说，当对一个线程，调用 interrupt() 时，
　　① 如果线程处于被阻塞状态（例如处于sleep, wait, join 等状态），那么线程将立即退出被阻塞状态，并抛出一个InterruptedException异常。仅此而已。
　　② 如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true，仅此而已。被设置中断标志的线程将继续正常运行，不受影响。

interrupt() 并不能真正的中断线程，需要被调用的线程自己进行配合才行。
也就是说，一个线程如果有被中断的需求，那么就可以这样做。
　　① 在正常运行任务时，经常检查本线程的中断标志位，如果被设置了中断标志就自行停止线程。
　　② 在调用阻塞方法时正确处理InterruptedException异常。（例如，catch异常后就结束线程。）
Thread.interrupted()清除标志位是为了下次继续检测标志位。
如果一个线程被设置中断标志后，选择结束线程那么自然不存在下次的问题，
而如果一个线程被设置中断标识后，进行了一些处理后选择继续进行任务，
而且这个任务也是需要被中断的，那么当然需要清除标志位了。

Future.cancel()：
Future 中的 cancel 方法用于取消任务的执行。它可以取消尚未开始执行的任务，也可以尝试取消正在执行的任务。
cancel 方法接收一个 mayInterruptIfRunning 参数，如果设置为 true，则表示在任务正在执行时尝试中断任务。如果任务已经开始执行，那么就会尝试中断线程来取消任务的执行。
如果任务已经完成或已经被取消，那么调用 cancel 方法将不会产生任何影响，并且方法会返回 false。如果成功取消了任务，那么方法会返回 true。
cancel 方法返回 true 表示任务被取消成功，返回 false 表示任务无法被取消，可能是因为任务已经完成或者已经被取消。
