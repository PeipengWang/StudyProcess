# Java的CAS

## CAS简介

CAS是一种无锁算法，该算法关键依赖两个值——期望值（旧值）和新值，底层CPU利用原子操作判断内存原值与期望值是否相等，如果相等就给内存地址赋新值，否则不做任何操作。
使用CAS进行无锁编程的步骤大致如下：
1、获得字段的期望值（oldValue）。
2、计算出需要替换的新值（newValue）。
3、通过CAS将新值（newValue）放在字段的内存地址上，如果CAS失败就重复第1步到第2步，一直到CAS成功，这种重复俗称CAS自旋。
4、当CAS将内存地址的值与预期值进行比较时，如果相等，就证明内存地址的值没有被修改，可以替换成新值，然后继续往下运行；如果不相等，就说明内存地址的值已经被修改，放弃替换操作，然后重新自旋。
其中具体实现在unsafe类中，实现方法是native类型

```
public final native boolean compareAndSwapObject(Object o, long offset, Object expected, Object update);
public final native boolean compareAndSwapInt(Object o, long offset, int expected, int update);
public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

最终操作系统层面的CAS是一条CPU的原子指令（cmpxchg指令），正是由于该指令具备原子性，因此使用CAS操作数据时不会造成数据不一致的问题，Unsafe提供的CAS方法直接通过native方式（封装C++代码）调用了底层的CPU指令cmpxchg。


下面先看一个不进行原子化导致的并发问题：

```
public class CASExample2 {

    private static int count;
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new CounterRunnable());
            thread.start();
        }
    }
    static class CounterRunnable implements Runnable{

        @Override
        public void run() {
            count = count + 1;
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            int newValue = count;
            System.out.println("Thread " + Thread.currentThread().getName() + ": Counter value = " + newValue);
        }
    }
}

```

这个实例中输出

```
Thread Thread-0: Counter value = 1
Thread Thread-1: Counter value = 4
Thread Thread-2: Counter value = 5
Thread Thread-3: Counter value = 5
Thread Thread-4: Counter value = 5
Thread Thread-5: Counter value = 6
Thread Thread-6: Counter value = 7
Thread Thread-7: Counter value = 8
Thread Thread-9: Counter value = 10
Thread Thread-8: Counter value = 10
```

这些线程按理说应该会从1输出到10，但是因为没有进行原子化产生了并发问题。
当然解决也比较简单，加个锁就可以了，但是在这里，我们将会使用AtomicInteger等原子类进行CAS操作

## 先简单加个锁试试

```
public class CASExample2 {

    private static int count;
    private static final  Object object = new Object();
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new CounterRunnable());
            thread.start();
        }
    }
    static class CounterRunnable implements Runnable{

        @Override
        public void run() {
           synchronized (object){
               count = count + 1;
               try {
                   Thread.sleep(1);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               int newValue = count;
               System.out.println("Thread " + Thread.currentThread().getName() + ": Counter value = " + newValue);
           }
        }
    }
}
```

## AtomicInteger等原子类进行CAS操作

据操作的目标数据类型，可以将JUC包中的原子类分为4类：基本原子类、数组原子类、原子引用类和字段更新原子类。
基本原子类的功能是通过原子方式更新Java基础类型变量的值。基本原子类主要包括以下三个：
AtomicInteger：整型原子类。
AtomicLong：长整型原子类。
AtomicBoolean：布尔型原子类。

数组原子类数组原子类的功能是通过原子方式更数组中的某个元素的值。数组原子类主要包括以下三个：
AtomicIntegerArray：整型数组原子类。
AtomicLongArray：长整型数组原子类。
AtomicReferenceArray：引用类型数组原子类。

引用原子类引用原子类主要包括以下三个：
AtomicReference：引用类型原子类。
AtomicMarkableReference：带有更新标记位的原子引用类型。
AtomicStampedReference：带有更新版本号的原子引用类型。
AtomicMarkableReference类将boolean标记与引用关联起来，可以解决使用AtomicBoolean进行原子更新时可能出现的ABA问题。AtomicStampedReference类将整数值与引用关联起来，可以解决使用AtomicInteger进行原子更新时可能出现的ABA问题。

字段更新原子类字段更新原子类主要包括以下三个：
AtomicIntegerFieldUpdater：原子更新整型字段的更新器。
AtomicLongFieldUpdater：原子更新长整型字段的更新器。
AtomicReferenceFieldUpdater：原子更新引用类型中的字段。

这里使用AtomicInteger进行了解。
创建AtomicInteger对象：

```
AtomicInteger atomicInteger = new AtomicInteger();
```

常用方法：

```
get()：获取当前的值。
set(int newValue)：设置新的值。
getAndSet(int newValue)：设置新的值，并返回旧值。
incrementAndGet()：自增并返回自增后的值。
decrementAndGet()：自减并返回自减后的值。
getAndIncrement()：返回当前值，并自增。
getAndDecrement()：返回当前值，并自减。
compareAndSet(int expect, int update)：如果当前值等于期望值expect，则将其更新为update。该方法返回一个布尔值，表示操作是否成功。
```

解决上述问题

```
import java.util.concurrent.atomic.AtomicInteger;

public class CASExample2 {

    private static AtomicInteger count = new AtomicInteger();
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(new CounterRunnable());
            thread.start();
        }
    }
    static class CounterRunnable implements Runnable{

        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName()+" into run");
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            int newValue = count.incrementAndGet();
            System.out.println("Thread " + Thread.currentThread().getName() + ": Counter value = " + newValue);
        }
    }
}
```

此时，实际上减少的锁定的位置，单纯的在int newValue = count.incrementAndGet();这里进行了原子化操作

```
Thread-0 into run
Thread-1 into run
Thread-2 into run
Thread-3 into run
Thread-4 into run
Thread-5 into run
Thread-6 into run
Thread-7 into run
Thread-8 into run
Thread-9 into run
Thread Thread-0: Counter value = 1
Thread Thread-1: Counter value = 2
Thread Thread-2: Counter value = 3
Thread Thread-3: Counter value = 4
Thread Thread-4: Counter value = 5
Thread Thread-5: Counter value = 6
Thread Thread-6: Counter value = 7
Thread Thread-7: Counter value = 8
Thread Thread-8: Counter value = 9
Thread Thread-9: Counter value = 10
```

## ABA问题

### 问题描述

ABA问题是指在并发编程中，当一个共享变量从初始值A经过一系列操作变为B并再次回到A时，如果其他线程在这期间对该共享变量进行读取和修改，就可能导致一些意外的结果。

简单来说，ABA问题在于无法区分共享变量的值是否在此期间被其他线程修改过。尽管变量的值回到了最初的状态A，但是在中间过程中可能出现了其他线程的干扰操作。
下面通过一个具体的示例来说明ABA问题：

假设有两个线程T1和T2同时对一个共享变量X进行操作，初始状态下，X的值为A。操作如下：

T1将X的值从A变为B。
T1将X的值从B变为A。
T2读取到X的值为A，并且做一些操作。
在上述过程结束后，T1再次读取到X的值为A。
在这个过程中，T1将变量X的值从A变为B再变回A，但是T2在中间阶段读取到的是A，然后做一些操作。这种情况下，T2可能无法察觉到变量X在过程中发生过变化，从而导致出现意外的结果。

ABA问题常见于使用CAS（Compare and Swap）等原子操作的并发程序中。CAS操作会先比较共享变量的值是否为预期值，如果是，则进行更新操作。但是对于ABA问题来说，即使CAS操作成功，也无法感知到中间是否有其他线程对共享变量进行了修改。

### 解决方法

使用版本号、引用更新等方法来解决ABA问题
AtomicStampedReference 是 java.util.concurrent.atomic 包中提供的一个类，用于解决 ABA 问题。ABA 问题指的是一个值原来是 A，变成了 B，然后又变回了 A。在某些并发场景下，如果只关注值的变化，可能无法感知到这个 ABA 问题。AtomicStampedReference 通过引入版本号的概念，能够检测到这种情况。
1、构造方法：

```
public AtomicStampedReference(V initialRef, int initialStamp);
```

initialRef：初始引用值。
initialStamp：初始版本号。
2、获取引用值：

```
public V getReference();
```

返回当前的引用值。
3、获取版本号：

```
public int getStamp();
```

返回当前的版本号。
4、比较并设置引用值和版本号：

```
public boolean compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp);
```

如果当前引用值等于 expectedReference 且当前版本号等于 expectedStamp，则将引用值更新为 newReference 和版本号更新为 newStamp，返回 true。否则，返回 false。
5、获取当前引用值和版本号，并更新版本号：

```
public boolean weakCompareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp);
```

类似于 compareAndSet，但是是弱一致性的操作。具体来说，如果在执行过程中引用值或版本号发生了变化，也会返回 true，但是不会保证写入一定成功。
6、设置引用值和版本号：

```
public void set(V newReference, int newStamp);
```

直接设置引用值和版本号。
7、尝试仅更新版本号：

```
public boolean attemptStamp(V expectedReference, int newStamp);
```

如果当前引用值等于 expectedReference，则尝试仅更新版本号为 newStamp。如果成功，返回 true；否则，返回 false。
8、获取当前引用值和版本号，并更新版本号：

```
public boolean compareAndSet(int[] expectedReference, int newReference, int[] expectedStamp, int newStamp);
```

通过一个整型数组来同时获取和更新引用值和版本号。数组的第一个元素是引用值，第二个元素是版本号。

```
import java.util.concurrent.atomic.AtomicStampedReference;

public class ABASolutionExample {
    private static AtomicStampedReference<String> atomicRef = new AtomicStampedReference<>("A", 0);
    public static void main(String[] args) {
        // 线程1对共享变量进行更新操作，将其从A变为B再变回A
        Thread thread = new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+",初始值:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
            int stamp = atomicRef.getStamp();
            String reference = atomicRef.getReference();
            System.out.println(stamp);
            System.out.println(reference);
            System.out.println("等待其他线程更新");
            boolean b = atomicRef.compareAndSet(reference, "B", stamp, stamp + 1);
            if(b){
                System.out.println(Thread.currentThread().getName()+",更新后:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
            }
            System.out.println(Thread.currentThread().getName()+",变回来。。。。， 当前值为:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
            stamp = atomicRef.getStamp();
            reference = atomicRef.getReference();
            atomicRef.compareAndSet(reference, "A", stamp, stamp+1);
            System.out.println(Thread.currentThread().getName()+",变回后的值:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
        });

        // 线程2对共享变量进行判断和更新操作
        Thread thread1 = new Thread(() -> {
            try {
                Thread.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
            int stamp = atomicRef.getStamp();
            String reference = atomicRef.getReference();
            System.out.println(stamp);
            System.out.println(reference);
            if(reference.equals("A")){
                atomicRef.compareAndSet(reference, "C", stamp, stamp + 1);
               stamp = atomicRef.getStamp();
               reference = atomicRef.getReference();
            }
            System.out.println(Thread.currentThread().getName());
            System.out.println(stamp);
            System.out.println(reference);
        });
        thread.start();
        thread1.start();

    }
}

```

```
A
等待其他线程更新
Thread-0,更新后:B，版本号为：1
Thread-0,变回来。。。。， 当前值为:B，版本号为：1
Thread-0,变回后的值:A，版本号为：2
Thread-1
2
A
Thread-1
3
C
```

可以看出，线程0进行了ABA变更后，线程1可以感知到线程0的版本变更的
稍微改变一下

```
package ABA;

import java.util.concurrent.atomic.AtomicStampedReference;

public class ABASolutionExample {
    private static AtomicStampedReference<String> atomicRef = new AtomicStampedReference<>("A", 0);
    public static void main(String[] args) {
        // 线程1对共享变量进行更新操作，将其从A变为B再变回A
        Thread thread = new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+",初始值:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
            int stamp = atomicRef.getStamp();
            String reference = atomicRef.getReference();
            System.out.println(stamp);
            System.out.println(reference);
            System.out.println("等待其他线程更新");
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //此时版本失效， 更新失败
            boolean b = atomicRef.compareAndSet(reference, "B", stamp, stamp + 1);
            if(b){
                System.out.println(Thread.currentThread().getName()+",更新后:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
            }
            System.out.println(Thread.currentThread().getName()+",变回来。。。。， 当前值为:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
            stamp = atomicRef.getStamp();
            reference = atomicRef.getReference();
            atomicRef.compareAndSet(reference, "A", stamp, stamp+1);
            System.out.println(Thread.currentThread().getName()+",变回后的值:" + atomicRef.getReference() + "，版本号为：" + atomicRef.getStamp());
        });

        // 线程2对共享变量进行判断和更新操作
        Thread thread1 = new Thread(() -> {
            try {
                Thread.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
            int stamp = atomicRef.getStamp();
            String reference = atomicRef.getReference();
            System.out.println(stamp);
            System.out.println(reference);
            if(reference.equals("A")){
                atomicRef.compareAndSet(reference, "C", stamp, stamp + 1);
               stamp = atomicRef.getStamp();
               reference = atomicRef.getReference();
            }
            System.out.println(Thread.currentThread().getName());
            System.out.println(stamp);
            System.out.println(reference);
        });
        thread.start();
        thread1.start();

    }
}

```

让线程0在更新为B之前等待10ms，线程1在修改之前等待5ms，这样线程1先更新为C，此时版本号变化，线程0就无法改变为B了，此时只能重新获取版本号来变化

```
Thread-0,初始值:A，版本号为：0
0
A
等待其他线程更新
Thread-1
0
A
Thread-1
1
C
Thread-0,变回来。。。。， 当前值为:C，版本号为：1
Thread-0,变回后的值:A，版本号为：2
```

## CAS进行并发控制

### 自旋锁

自旋锁是一种基于循环等待的锁策略，线程在申请锁时如果发现其它线程已经持有该锁，就会进入忙等待的状态，不释放CPU时间片，而是进行自旋等待直到获取到锁为止。使用CAS实现自旋锁的核心思想是，通过原子操作对一个共享变量进行比较和交换，来判断是否成功获取锁。
基本原理是，申请锁的线程通过CAS操作尝试将锁的状态从未锁定状态改为锁定状态，如果操作成功，表示当前线程获取到了锁；否则，表示锁已经被其他线程持有，申请线程会不断尝试CAS操作，直到获取到锁为止。
如下实例：

```
import java.util.concurrent.atomic.AtomicInteger;

public class SpinLock{
    private AtomicInteger stat = new AtomicInteger(0);
    public void lock(){
        while (!stat.compareAndSet(0,1)){

        }
    }
    public void unlock(){
        stat.set(0);
    }
}
```

SpinLock类使用AtomicInteger实现自旋锁。state变量用于表示锁的状态，初始值为0，表示未锁定状态。

在lock()方法中，使用compareAndSet()方法进行原子比较和交换操作。如果state的值为0，则将其更新为1，表示当前线程成功获取到了锁。如果compareAndSet()返回false，表示锁已经被其他线程持有，当前线程会一直循环等待，直到获取到锁为止。
在unlock()方法中，将state的值设置回0，表示释放锁。

实例:

```
public class SpinLockExample {
    private SpinLock spinLock = new SpinLock();
    private int count = 0;
    public void increment(){
        spinLock.lock();
        count++;
        spinLock.unlock();

    }
    public static void main(String[] args) {
        SpinLockExample example = new SpinLockExample();
        int numThreads = 100;
        Thread[] threads = new Thread[numThreads];

        for (int i = 0; i < numThreads; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    example.increment();
                }
            });
            threads[i].start();
        }

        // 等待所有线程执行完毕
        for (int i = 0; i < numThreads; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("Count: " + example.count);
    }

}

```

Example类使用了SpinLock来保护共享变量count的操作。创建了10个线程，并且每个线程对count进行1000次递增操作。最后输出count的值。
这个例子展示了如何使用CAS实现自旋锁来保护共享数据，确保线程安全性。

### 无锁算法

无锁算法是一种无需使用锁来实现同步的并发编程技术，它通过使用原子操作或无锁数据结构，使得多个线程可以在不互斥的情况下并发地访问共享数据。
基本原理是，多个线程之间对共享数据进行操作时，采用原子操作或无锁数据结构，避免使用传统的锁机制。通过使用CAS等原子操作，同时保证数据的一致性和并发性，而无需使用互斥锁来保护共享数据。在无锁算法中，不会出现线程阻塞等待锁释放的情况，所有线程都可以并发地执行操作。
无锁算法的优势在于减少了锁带来的开销，提高了并发性能，同时也降低了死锁和饥饿问题的可能性。然而，无锁算法也带来了额外的复杂性，需要对并发操作进行仔细设计和调试，确保数据的一致性和安全性。

```
import java.util.concurrent.atomic.AtomicInteger;

public class Counter {
    private AtomicInteger value = new AtomicInteger(0); // 共享计数器

    public void increment() {
        int currentValue;
        do {
            currentValue = value.get(); // 获取当前值
        } while (!value.compareAndSet(currentValue, currentValue + 1)); // CAS操作增加计数
    }

    public int getValue() {
        return value.get();
    }
}

```

在上面的示例中，Counter类使用AtomicInteger实现了一个无锁的计数器。value变量为共享的计数器，初始值为0。
在increment()方法中，使用do-while循环结构，不断尝试使用compareAndSet()方法来原子地更新计数器的值。首先获取当前的计数器值，然后使用compareAndSet()进行比较和交换操作，如果当前值与获取到的值相等，那么将其增加1并更新计数器的值。如果compareAndSet()返回false，表示其他线程已经修改了计数器的值，当前线程会重复这个过程直到成功为止。
在getValue()方法中，直接返回计数器的当前值。

```
public class Example {
    private Counter counter = new Counter();

    public void execute() {
        for (int i = 0; i < 1000; i++) {
            counter.increment(); // 对计数器进行递增操作
        }
    }

    public static void main(String[] args) {
        Example example = new Example();
        int numThreads = 10;
        Thread[] threads = new Thread[numThreads];

        for (int i = 0; i < numThreads; i++) {
            threads[i] = new Thread(example::execute);
            threads[i].start();
        }

        // 等待所有线程执行完毕
        for (int i = 0; i < numThreads; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("Value: " + example.counter.getValue());
    }
}

```

在上面的示例中，Example类使用了Counter来实现无锁的计数器。创建了10个线程，每个线程执行execute()方法，对计数器进行1000次递增操作。最后输出计数器的值。

### CAS在无锁算法、并发编程中的应用实例

无锁算法的应用实例：
a. 无锁队列（Lock-Free Queue）：CAS可以用于实现无锁队列，实现线程安全的入队（Enqueue）和出队（Dequeue）操作。多个线程可以同时进行入队和出队操作，而不需要使用传统的锁机制。通过使用CAS来更新队列的头指针和尾指针，线程可以通过自旋重试来保证操作的原子性。

b. 无锁哈希表：CAS可以用于实现无锁哈希表，允许多个线程同时访问和修改哈希表中的数据。通过使用CAS来更新哈希表中的节点指针，线程可以并发地进行插入、删除和查找操作，避免了传统锁带来的竞争和串行化问题。

并发编程中的应用实例：
a. 状态管理：CAS可用于实现状态管理，例如标志位的设置、重置和检查操作。多个线程可以通过CAS操作来检查和修改共享标志位，从而实现并发状态的同步和控制。

b. 计数器：CAS可以用于实现并发计数器，允许多个线程同时增加或减少计数器的值。通过使用CAS来原子地更新计数器的值，线程可以并发地对计数器进行操作，避免了传统锁带来的互斥访问和串行化问题。

c. 数据结构的更新：CAS可以用于实现并发数据结构的更新操作。例如，在跳表（Skip List）中，CAS可用于插入、删除和修改节点的操作，确保操作的原子性和线程安全性。

## 高并发下的CAS性能

在争用激烈的场景下，会导致大量的CAS空自旋。大量的CAS空自旋会浪费大量的CPU资源，大大降低了程序的性能。
Java 8提供了一个新的类LongAdder，以空间换时间的方式提升高并发场景下CAS操作的性能。

LongAdder的核心思想是热点分离，与ConcurrentHashMap的设计思想类似：将value值分离成一个数组，当多线程访问时，通过Hash算法将线程映射到数组的一个元素进行操作；而获取最终的value结果时，则将数组的元素求和。
最终，通过LongAdder将内部操作对象从单个value值“演变”成一系列的数组元素，从而减小了内部竞争的粒度。

## CAS的弊端

CAS操作的弊端主要有以下三点：
**ABA问题**
JDK提供了两个类AtomicStampedReference和AtomicMarkableReference来解决ABA问题。
**只能保证一个共享变量之间的原子性操作**
JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个AtomicReference实例后再进行CAS操作。
**开销问题**
解决CAS恶性空自旋的有效方式之一是以空间换时间，较为常见的方案为：
分散操作热点，使用LongAdder替代基础原子类AtomicLong，LongAdder将单个CAS热点（value值）分散到一个cells数组中。
使用队列削峰，将发生CAS争用的线程加入一个队列中排队，降低CAS争用的激烈程度。JUC中非常重要的基础类AQS（抽象队列同步器）就是这么做的。


1、性能：
CAS操作是原子操作，不需要进入内核态，因此它的执行速度比传统锁要快。CAS在硬件级别使用了原子指令，避免了线程的上下文切换和内核态的开销。
传统锁通常涉及线程的阻塞和唤醒操作，这需要线程从用户态切换到内核态，开销较大。当线程竞争激烈时，频繁的上锁和解锁操作会导致性能下降。
2、可伸缩性：
CAS操作是乐观并发控制的一种形式，不需要对共享资源进行独占性访问，因此具有良好的可伸缩性。多个线程可以同时读或尝试更新共享变量的值，而不会相互阻塞。
传统锁在某个线程持有锁时会阻塞其他线程的访问，这可能导致线程之间的竞争和争用，影响可伸缩性。如果使用粒度过大的锁或者热点数据访问，则会限制并发性，降低系统的可伸缩性。
3、CAS也有一些限制和适用条件：
CAS操作能够保证原子性，但无法解决ABA问题（某个值先变为A，后又变回原来的A，那么在CAS检查时可能无法识别出这种变化）。为了解决ABA问题，可以使用带有版本号或时间戳的CAS。
CAS操作适合在高度竞争的情况下使用，当竞争不激烈时，使用传统锁可以更好地处理。
CAS操作只能针对单个变量的原子操作，无法实现复杂的同步需求。而传统锁可以支持更复杂的同步机制，如读写锁、悲观锁等。


引用自，详细请进入链接学习：
https://github.com/Loserfromlazy/Code_Career/blob/master
https://blog.csdn.net/u012581020/article/details/132165800
https://pdai.tech/md/java/thread/java-thread-x-juc-AtomicInteger.html






