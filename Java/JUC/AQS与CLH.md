# AQS与CLH

CLH 锁是对自旋锁的一种改良  
互斥：多线程竞争环境下，同一时刻只有一个线程能访问  
同步：互斥的基础上，保证线程运行的有序性  
三个机制：  
状态  state  
队列  CLH  
waitstate  
操作：  
    1、抢位置  
        1）、这个位置是空的  state=0  CAS  保证只有一个线程抢到锁  
        2）、重复抢   判断当前线程是不是已经持有所的线程，记录锁的次数（知道释放锁的时候应该释放多少次）  
        3）、这个锁已经被抢占  入队、堵塞、等待唤醒、继续运行、唤醒其他线程。  
    2、入队  头插法，尾插法，  addWait， enq  
    3、堵塞，等待  
    4、出队（唤醒）  
    5、释放锁  
    
CLH算法  
MCS算法  


方法：
class Node  
Node，它用于构建基于链表的队列中的节点。这些节点被用于构建并管理同步器（例如AbstractQueuedSynchronizer）中的等待队列。  
enq(AbstractQueuedSynchronizer.Node node)：方法用于将一个节点添加到队列的尾部，并返回队列中的最后一个节点。如果队列为空，则会创建一个新节点作为头节点，并将其同时作为尾节点。如果队列不为空，则将新节点添加到尾节点后，并将新节点设为尾节点。这个方法在一个循环中执行，直到成功添加节点为止。  

addWaiter(AbstractQueuedSynchronizer.Node mode)：方法用于创建一个新的节点，并尝试将其添加到队列的尾部。如果队列不为空，则将新节点添加到尾节点后；如果队列为空，则会先创建一个头节点，然后再添加新节点到尾节点后。如果添加新节点失败，则会调用 enq 方法将其添加到队列中。最终，这个方法返回新创建的节点。  

unparkSuccessor(AbstractQueuedSynchronizer.Node node) ：这个方法用于唤醒队列中的后继节点，以便它们可以尝试获取锁。  
doReleaseShared()：用于释放共享模式下的锁。  
setHeadAndPropagate(AbstractQueuedSynchronizer.Node node, int propagate) ：用于设置新的头节点，并在适当的情况下传播唤醒信号。  
cancelAcquire(AbstractQueuedSynchronizer.Node node)：这个方法用于取消节点对资源的获取，并对相应的等待队列进行调整。  
 shouldParkAfterFailedAcquire(AbstractQueuedSynchronizer.Node pred, AbstractQueuedSynchronizer.Node node) ：这个方法用于确定在获取资源失败后，当前线程是否应该被阻塞。



线程的状态：中断  
