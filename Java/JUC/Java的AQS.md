# Java的AQS

AbstractQueuedSynchronizer抽象类是核心，需要重点掌握。它提供了一个基于FIFO队列，可以用于构建锁或者其他相关同步装置的基础框架。

AQS的核心思想是，使用一个整型的状态（state）来表示被保护的资源的状态，并通过CAS（Compare and Swap）操作来进行原子性地状态更新。AQS定义了两种资源共享方式：独占（Exclusive）和共享（Shared），并提供了相应的获取（acquire）和释放（release）操作。
以下是AQS的一些关键方法和概念：
getState方法：获取当前同步器的状态。
setState方法：设置当前同步器的状态。
compareAndSetState方法：使用CAS操作设置同步器的状态。
acquire方法：用于获取共享资源，它是一个模板方法，具体的实现在子类中定义。
钩子方法：
tryAcquire：尝试获取资源，成功返回true，失败返回false。
release方法：用于释放共享资源，同样是一个模板方法，具体的实现在子类中定义。
  钩子方法：tryRelease：尝试释放资源。
acquireShared方法：用于获取共享资源，也是一个模板方法。
  钩子方法：tryAcquireShared：尝试获取共享资源。
releaseShared方法：用于释放共享资源，同样是一个模板方法。
  钩子方法：tryReleaseShared：尝试释放共享资源。
通过继承AQS并实现其中的钩子方法，开发者可以相对容易地构建出符合自己需求的同步工具。常见的基于AQS的同步器包括ReentrantLock、ReentrantReadWriteLock、Semaphore、CountDownLatch等。

