从ReentrantLock的实现看AQS的原理及应用
https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html

对AQS的简单理解及自定义锁的实现
https://juejin.cn/post/6844904163382411278

ReentrantLock源码分析
https://pdai.tech/md/java/thread/java-thread-x-lock-ReentrantLock.html


ReetrantLock实际上是通过定义Sync接口，Sync接口两个实现NonfairSync和FairSync来定义公平锁和非公平锁

公平锁与非公平锁实现原理是是否检查当前线程恰好完成，非公平锁会直接持有这个锁，公平锁会加到队列尾部

AQS简单来说是一个CLH双端队列+state变量来维护锁状态的

CLH双端队列里面存储Node节点，代表当前线程的状态。

state是voltile类型的变量，在AQS中通过检查state变量的值来确定锁状态

锁分为独占锁和共享锁

我们可以通过修改State字段表示的同步状态来实现多线程的独占模式和共享模式（加锁过程）。

![img](https://p0.meituan.net/travelcube/27605d483e8935da683a93be015713f331378.png)

![img](https://p0.meituan.net/travelcube/3f1e1a44f5b7d77000ba4f9476189b2e32806.png)


至于锁的可重入机制，实际上是检查当前持有锁线程的Node是否是当前线程，如果是则可进入。

