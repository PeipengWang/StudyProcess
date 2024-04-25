参考来源：https://juejin.cn/post/7000515796053000228
# Java队列--LinkedBlockingQueue
LinkedBlockingQueue是基于单向链表的阻塞队列，先进先出的顺序，支持多线程并发操作。LinkedBlockingQueue可认为是无界队列，多用于任务队列。  
LinkedBlockingQueue 常用于多线程的生产者-消费者场景，其中生产者线程将数据放入队列，而消费者线程从队列中取出数据。它还用于线程池的任务队列等。  
## 定义方法  
无参定义，默认构造方法，队列容量为Integer.MAX_VALUE  
```
public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
```
指定队列容量的构造方法
```
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }
```
基于集合构建队列，默认容量为Integer.MAX_VALUE
```
    public LinkedBlockingQueue(Collection<? extends E> c) {
        this(Integer.MAX_VALUE);
        final ReentrantLock putLock = this.putLock;
        putLock.lock(); // Never contended, but necessary for visibility
        try {
            int n = 0;
            for (E e : c) {
                if (e == null)
                    throw new NullPointerException();
                if (n == capacity)
                    throw new IllegalStateException("Queue full");
                enqueue(new Node<E>(e));
                ++n;
            }
            count.set(n);
        } finally {
            putLock.unlock();
        }
    }
```
可以看到，第一第三种方式都是设置大小为Integer.MAX_VALUE，这容易导致堆栈溢出，因此一般建议使用自定义大小的方式来定义  
## 基本属性
在LinkedBlockingQueue中定义的变量及其含义如下：  
capacity：该变量表示队列的容量，设置该值则变为一个有界队列；如果不设置的话默认取值为Integer.MAX_VALUE，也可以认为是无界队列  
count：当前队列中元素的数量  
head和last：分别表示链表的头尾节点，其中头结点head不存储元素，head.item = null  
takeLock和notEmpty：出队的锁以及出队条件  
putLock和notFull：入队的锁以及入队条件  
可以看出与ArrayBlockingQueue不同的是，在LinkedBlockingQueue中，入队和出队分别使用两个锁，两个锁可以分别认为是读写锁和读锁，这里的具体原因在后面会进行详细描述
## 基本使用  
```
        //定义
        LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<>();
        // 入队方法
        queue.add(1);
        queue.offer(2);
        queue.offer(3,10, TimeUnit.SECONDS);
        queue.put(4);
        System.out.println(queue.size());
        // 出队方法
        Integer x1 = queue.remove();
        Integer x2 = queue.poll();
        Integer x3 = queue.poll(10, TimeUnit.SECONDS);
        Integer x4 = queue.take();
        System.out.println(x1);
        System.out.println(x2);
        System.out.println(x3);
        System.out.println(x4);
        //队列已用数目
        System.out.println(queue.size());
        //剩余可用数目
        System.out.println(queue.remainingCapacity());
```
## 入队和出队方法  
### 入队  
#### put  
首先我们来看下阻塞入队方法put(e)的实现原理，代码如下：  
```
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    // 入队锁上锁
    putLock.lockInterruptibly();
    try {
    	// 如果队列中元素的数量等于队列的容量，则阻塞当前线程
        while (count.get() == capacity) {
            notFull.await();
        }
        // 执行入队操作
        enqueue(node);
        // 元素数量增1，返回操作前的数量
        c = count.getAndIncrement();
        // c+1为当前队列的元素，如果小于容量，则唤醒notFull的等待线程，触发继续入队操作
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
    	// 释放锁
        putLock.unlock();
    }
    // c为入队前的元素数量，也就是入队前队列为空，则需要唤醒非空条件notEmpty的等待线程，触发出队操作
    if (c == 0)
        signalNotEmpty();
}

private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
    	// 唤醒出队等待的线程
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}

```
通过上面的代码，我们可以看出put(e)方法的主要流程如下：  

首先生成待插入节点Node<E> node = new Node<E>(e)  
然后尝试使用putLock上锁：如果当前无其他线程进行入队操作，则上锁成功；如果当前有其他线程进行入队操作，则进行等待，直到加锁成功；  
加锁成功之后，首先进行队列容量的检查：如果元素数量等于队列容量，则无空间插入新元素，那么调用notFull.await()阻塞当前线程（当前线程被加入notFull条件的等待队列中；如果当前线程被唤醒，也需要再次检查是否有空间插入，如果没有还需要继续等待；  
当队列有空间时，则调用enqueue(node)进行入队操作，将新节点node插入到链表中  
入队完成后，对元素数量进行+1操作，并获取入队前的元素数量  
判断当前元素数量小于队列容量时，则调用notFull.signal()唤醒一个等待入队的线程  
释放锁  
最后检查入队前元素数量为0，也就是队列为空时，那么此时队列不为空，则需要唤醒等待出队条件notEmpty的线程，触发出队操作，调用方法signalNotEmpty。  
到这里我们已经看完了整个put(e)操作的主流程，然后我们再看下enqueue(node)操作的具体逻辑，代码如下：  
```
private void enqueue(Node<E> node) {
    // assert putLock.isHeldByCurrentThread();
    // assert last.next == null;
    last = last.next = node;
}
```
#### 入队操作offer
在LinkedBlockingQueue中提供了两个offer重载方法，一个是offer(E e)，另外一个是offer(E e, long timeout, TimeUnit unit)，两者的区别如下：  
offer(E e)方法在入队时，如果当前队列有空间则直接入队，没有空间则入队失败，返回false；  
offer(E e, long timeout, TimeUnit unit)是带等待时间的阻塞入队方法，有空间直接入队，没有空间则等待特定的时间，如果依然无法入队，则返回false  
#### 入队操作add  
在LinkedBlockingQueue中，由于继承了AbstractQueue类，所以add方法也是使用的AbstractQueue中的定义，代码如下；add方法直接调用了offer(E e)方法，并判断是否入队成功，如果入队失败则抛出异常  
### 出队  
#### 出队操作take  
我们还是先看阻塞出队方法take()的实现。  
```
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    // 加锁
    takeLock.lockInterruptibly();
    try {
        // 判断队列容量，如果为空则等待
        while (count.get() == 0) {
            notEmpty.await();
        }
        // 出队操作
        x = dequeue();
        // 队列元素数量-1，返回
        c = count.getAndDecrement();
        // 出队前队列元素大于1，也就是当前队列还有元素，则唤醒一个等待出队的线程
        if (c > 1)
            notEmpty.signal();
    } finally {
        // 释放锁
        takeLock.unlock();
    }
    // 出队前队列元素等于队列容量，也就是出队后队列不满，则唤醒等待入队的线程
    if (c == capacity)
        signalNotFull();
    return x;
}

private void signalNotFull() {
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        // 唤醒等待入队的线程
        notFull.signal();
    } finally {
        putLock.unlock();
    }
}
```
通过上面的代码，我们可以看出take()方法的主要流程如下：  

尝试使用takeLock上锁：如果当前无其他线程进行出队操作，则上锁成功；如果当前有其他线程进行出队操作，则进行等待，直到加锁成功；  
加锁成功之后，首先进行队列容量的检查：如果队列为空，则调用notEmpty.await()阻塞当前线程；如果当前线程被唤醒，也需要再次检查队列是否为空，如果为空则继续等待；  
当队列不为空，则调用dequeue()进行出队操作，返回出队元素x；  
出队完成后，对元素数量进行-1操作，并获取出队前的元素数量  
判断当前队列中是否还有元素，如果有则调用notEmpty.signal()唤醒一个等待出队的线程  
释放锁  
最后检查出队前队列是否满的，如果是满的，则出队后队列不满，则需要唤醒等待入队条件notFull的线程，触发入队操作，调用方法signalNotFull  
在进行出队操作时，调用dequeue()方法，下面看下该方法的具体实现，代码如下：  
```
private E dequeue() {
    // assert takeLock.isHeldByCurrentThread();
    // assert head.item == null;
    // 当前头结点（头结点不存储数据，第一个元素为head.next）
    Node<E> h = head;
    // 当前队列中第一个元素
    Node<E> first = h.next;
    // 原头结点设置无效
    h.next = h; // help GC
    // 最新的头结点指向第一个元素first
    head = first;
    // 获得第一个元素的值
    E x = first.item;
    // 将第一个元素值设置为null，第一个元素变成头结点
    first.item = null;
    // 返回第一个元素值
    return x;
}

```
#### 出队操作poll  
同样的，LinkedBlockingQueue也提供了两个出队poll方法，一个是poll()，有元素则直接出队，无元素则返回null；另一个是poll(long time, TimeUnit unit)，带等待时间的出队方法，当有元素时直接出队，无元素时则等待特定时间。  
#### 出队操作remove
在LinkedBlockingQueue中，remove方法也是直接使用的父类AbstractQueue中的remove方法；方法直接调用了poll()方法，如果出队成功则返回出队元素，出队失败则抛出NoSuchElementException异常。  

## 使用实例
生产者：  
```
// 生产者线程
Thread producer = new Thread(() -> {
    try {
           queue.put("Item 1");
           queue.put("Item 2");
            // 队列已满，put操作将被阻塞
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
producer.start();
```
消费者：  
```
// 消费者线程
Thread consumer = new Thread(() -> {
    try {
        String item = queue.take();
        System.out.println("Consumed: " + item);
        Thread.sleep(1000);
        item = queue.take();
        System.out.println("Consumed: " + item);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
consumer.start();
```
##  对比ArrayBlockingQueue  
ArrayBlockingQueue中，使用了一个ReentrantLock lock作为入队和出队的锁，并使用两个条件notEmpty和notFull来进行线程间通信。而在本文介绍的LinkedBlockingQueue中，使用了两个锁putLock和takeLock分别作为入队和出队的锁，同样使用了两个锁的两个条件notFull和notEmpty进行线程间通信。  
由于在ArrayBlockingQueue中，入队和出队操作共用了同一个锁，所以两个操作之间会有相互影响；而在LinkedBlockingQueue中，入队和出队操作分别使用不同的锁，则入队和出队互不影响，可以提供队列的操作性  



参考来源：https://juejin.cn/post/7000515796053000228






