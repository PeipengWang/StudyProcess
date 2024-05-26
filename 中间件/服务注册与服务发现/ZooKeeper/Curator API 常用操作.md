# Curator详解



zookeeper不是为高可用性设计的，但它使用`ZAB`协议达到了极高的一致性。所以它经常被选作注册中心、配置中心、分布式锁等场景。

它的性能是非常有限的，而且API并不是那么好用。xjjdog倾向于使用基于`Raft`协议的`Etcd`或者`Consul`，它们更加轻量级一些。

Curator是netflix公司开源的一套zookeeper客户端，目前是Apache的顶级项目。与Zookeeper提供的原生客户端相比，Curator的抽象层次更高，简化了Zookeeper客户端的开发量。Curator解决了很多zookeeper客户端非常底层的细节开发工作，包括连接重连、反复注册wathcer和NodeExistsException 异常等。

Curator由一系列的模块构成，对于一般开发者而言，常用的是curator-framework和curator-recipes，下面对此依次介绍。

## 1.maven依赖

最新版本的curator 4.3.0支持zookeeper 3.4.x和3.5，但是需要注意curator传递进来的依赖，需要和实际服务器端使用的版本相符，以我们目前使用的zookeeper 3.4.6为例。

```
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.3.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.3.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.6</version>
</dependency>
```

## 2.curator-framework

下面是一些常见的zk相关的操作。

```java
ublic static CuratorFramework getClient() {
    return CuratorFrameworkFactory.builder()
            .connectString("127.0.0.1:2181")
            .retryPolicy(new ExponentialBackoffRetry(1000, 3))
            .connectionTimeoutMs(15 * 1000) //连接超时时间，默认15秒
            .sessionTimeoutMs(60 * 1000) //会话超时时间，默认60秒
            .namespace("arch") //设置命名空间
            .build();
}
 
public static void create(final CuratorFramework client, final String path, final byte[] payload) throws Exception {
    client.create().creatingParentsIfNeeded().forPath(path, payload);
}
 
public static void createEphemeral(final CuratorFramework client, final String path, final byte[] payload) throws Exception {
    client.create().withMode(CreateMode.EPHEMERAL).forPath(path, payload);
}
 
public static String createEphemeralSequential(final CuratorFramework client, final String path, final byte[] payload) throws Exception {
    return client.create().withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(path, payload);
}
 
public static void setData(final CuratorFramework client, final String path, final byte[] payload) throws Exception {
    client.setData().forPath(path, payload);
}
 
public static void delete(final CuratorFramework client, final String path) throws Exception {
    client.delete().deletingChildrenIfNeeded().forPath(path);
}
 
public static void guaranteedDelete(final CuratorFramework client, final String path) throws Exception {
    client.delete().guaranteed().forPath(path);
}
 
public static String getData(final CuratorFramework client, final String path) throws Exception {
    return new String(client.getData().forPath(path));
}
 
public static List<String> getChildren(final CuratorFramework client, final String path) throws Exception {
    return client.getChildren().forPath(path);
}

```

## 3.curator-recipes

curator-recipes 提供了一些zk的典型使用场景的参考。下面主要介绍一下开发中常用的组件。

### 事件监听

zookeeper原生支持通过注册watcher来进行事件监听，但是其使用不是特别方便，需要开发人员自己反复注册watcher，比较繁琐。

Curator引入Cache来实现对zookeeper服务端事务的监听。Cache是Curator中对事件监听的包装，其对事件的监听其实可以近似看作是一个本地缓存视图和远程Zookeeper视图的对比过程。同时，Curator能够自动为开发人员处理反复注册监听，从而大大简化原生api开发的繁琐过程。

#### 1）Node Cache

```java
public static void nodeCache() throws Exception {
    final String path = "/nodeCache";
    final CuratorFramework client = getClient();
    client.start();
 
    delete(client, path);
    create(client, path, "cache".getBytes());
 
    final NodeCache nodeCache = new NodeCache(client, path);
    nodeCache.start(true);
    nodeCache.getListenable()
            .addListener(() -> System.out.println("node data change, new data is " + new String(nodeCache.getCurrentData().getData())));
 
    setData(client, path, "cache1".getBytes());
    setData(client, path, "cache2".getBytes());
 
    Thread.sleep(1000);
 
    client.close();
}
```

NodeCache可以监听指定的节点，注册监听器后，节点的变化会通知相应的监听器

#### 2）Path Cache

Path Cache 用来监听ZNode的子节点事件，包括added、updateed、removed，Path Cache会同步子节点的状态，产生的事件会传递给注册的PathChildrenCacheListener。

```java
public static void pathChildrenCache() throws Exception {
        final String path = "/pathChildrenCache";
        final CuratorFramework client = getClient();
        client.start();
 
        final PathChildrenCache cache = new PathChildrenCache(client, path, true);
        cache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT);
    cache.getListenable().addListener((client1, event) -> {
            switch (event.getType()) {
                case CHILD_ADDED:
                    System.out.println("CHILD_ADDED:" + event.getData().getPath());
                    break;
                case CHILD_REMOVED:
                    System.out.println("CHILD_REMOVED:" + event.getData().getPath());
                    break;
                case CHILD_UPDATED:
                    System.out.println("CHILD_UPDATED:" + event.getData().getPath());
                    break;
                case CONNECTION_LOST:
                    System.out.println("CONNECTION_LOST:" + event.getData().getPath());
                    break;
                case CONNECTION_RECONNECTED:
                    System.out.println("CONNECTION_RECONNECTED:" + event.getData().getPath());
                    break;
                case CONNECTION_SUSPENDED:
                    System.out.println("CONNECTION_SUSPENDED:" + event.getData().getPath());
                    break;
                case INITIALIZED:
                    System.out.println("INITIALIZED:" + event.getData().getPath());
                    break;
                default:
                    break;
            }
        });
 
//        client.create().withMode(CreateMode.PERSISTENT).forPath(path);
        Thread.sleep(1000);
 
        client.create().withMode(CreateMode.PERSISTENT).forPath(path + "/c1");
        Thread.sleep(1000);
 
        client.delete().forPath(path + "/c1");
        Thread.sleep(1000);
 
        client.delete().forPath(path); //监听节点本身的变化不会通知
        Thread.sleep(1000);
 
        client.close();
    }
```

#### 3）Tree Cache

Path Cache和Node Cache的“合体”，监视路径下的创建、更新、删除事件，并缓存路径下所有孩子结点的数据。

```java
public static void treeCache() throws Exception {
    final String path = "/treeChildrenCache";
    final CuratorFramework client = getClient();
    client.start();
 
    final TreeCache cache = new TreeCache(client, path);
    cache.start();
 
    cache.getListenable().addListener((client1, event) -> {
        switch (event.getType()){
            case NODE_ADDED:
                System.out.println("NODE_ADDED:" + event.getData().getPath());
                break;
            case NODE_REMOVED:
                System.out.println("NODE_REMOVED:" + event.getData().getPath());
                break;
            case NODE_UPDATED:
                System.out.println("NODE_UPDATED:" + event.getData().getPath());
                break;
            case CONNECTION_LOST:
                System.out.println("CONNECTION_LOST:" + event.getData().getPath());
                break;
            case CONNECTION_RECONNECTED:
                System.out.println("CONNECTION_RECONNECTED:" + event.getData().getPath());
                break;
            case CONNECTION_SUSPENDED:
                System.out.println("CONNECTION_SUSPENDED:" + event.getData().getPath());
                break;
            case INITIALIZED:
                System.out.println("INITIALIZED:" + event.getData().getPath());
                break;
            default:
                break;
        }
    });
 
    client.create().withMode(CreateMode.PERSISTENT).forPath(path);
    Thread.sleep(1000);
 
    client.create().withMode(CreateMode.PERSISTENT).forPath(path + "/c1");
    Thread.sleep(1000);
 
    setData(client, path, "test".getBytes());
    Thread.sleep(1000);
 
    client.delete().forPath(path + "/c1");
    Thread.sleep(1000);
 
    client.delete().forPath(path);
    Thread.sleep(1000);
 
    client.close();
}
```

### 选举

curator提供了两种方式，分别是Leader Latch和Leader Election。

#### 1）Leader Latch

随机从候选着中选出一台作为leader，选中之后除非调用close()释放leadship，否则其他的后选择无法成为leader

```java
public class LeaderLatchTest {
 
    private static final String PATH = "/demo/leader";
 
    public static void main(String[] args) {
        List<LeaderLatch> latchList = new ArrayList<>();
        List<CuratorFramework> clients = new ArrayList<>();
        try {
            for (int i = 0; i < 10; i++) {
                CuratorFramework client = getClient();
                client.start();
                clients.add(client);
 
                final LeaderLatch leaderLatch = new LeaderLatch(client, PATH, "client#" + i);
                leaderLatch.addListener(new LeaderLatchListener() {
                    @Override
                    public void isLeader() {
                        System.out.println(leaderLatch.getId() + ":I am leader. I am doing jobs!");
                    }
 
                    @Override
                    public void notLeader() {
                        System.out.println(leaderLatch.getId() + ":I am not leader. I will do nothing!");
                    }
                });
                latchList.add(leaderLatch);
                leaderLatch.start();
            }
            Thread.sleep(1000 * 60);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            for (CuratorFramework client : clients) {
                CloseableUtils.closeQuietly(client);
            }
 
            for (LeaderLatch leaderLatch : latchList) {
                CloseableUtils.closeQuietly(leaderLatch);
            }
        }
    }
 
    public static CuratorFramework getClient() {
        return CuratorFrameworkFactory.builder()
                .connectString("127.0.0.1:2181")
                .retryPolicy(new ExponentialBackoffRetry(1000, 3))
                .connectionTimeoutMs(15 * 1000) //连接超时时间，默认15秒
                .sessionTimeoutMs(60 * 1000) //会话超时时间，默认60秒
                .namespace("arch") //设置命名空间
                .build();
    }
 
}
```

#### 2）Leader Election

通过LeaderSelectorListener可以对领导权进行控制， 在适当的时候释放领导权，这样每个节点都有可能获得领导权。 而LeaderLatch则一直持有leadership， 除非调用close方法，否则它不会释放领导权。

```java
public class LeaderSelectorTest {
    private static final String PATH = "/demo/leader";
 
    public static void main(String[] args) {
        List<LeaderSelector> selectors = new ArrayList<>();
        List<CuratorFramework> clients = new ArrayList<>();
        try {
            for (int i = 0; i < 10; i++) {
                CuratorFramework client = getClient();
                client.start();
                clients.add(client);
 
                final String name = "client#" + i;
                LeaderSelector leaderSelector = new LeaderSelector(client, PATH, new LeaderSelectorListenerAdapter() {
                    @Override
                    public void takeLeadership(CuratorFramework client) throws Exception {
                        System.out.println(name + ":I am leader.");
                        Thread.sleep(2000);
                    }
                });
 
                leaderSelector.autoRequeue();
                leaderSelector.start();
                selectors.add(leaderSelector);
            }
            Thread.sleep(Integer.MAX_VALUE);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            for (CuratorFramework client : clients) {
                CloseableUtils.closeQuietly(client);
            }
 
            for (LeaderSelector selector : selectors) {
                CloseableUtils.closeQuietly(selector);
            }
 
        }
    }
 
    public static CuratorFramework getClient() {
        return CuratorFrameworkFactory.builder()
                .connectString("127.0.0.1:2181")
                .retryPolicy(new ExponentialBackoffRetry(1000, 3))
                .connectionTimeoutMs(15 * 1000) //连接超时时间，默认15秒
                .sessionTimeoutMs(60 * 1000) //会话超时时间，默认60秒
                .namespace("arch") //设置命名空间
                .build();
    }
 
}

```

### 分布式锁

#### 0）Zookeeper分布式锁原理

- 核心思想：当客户端要获取锁，则创建节点，使用完锁就删除该节点

流程：

- 客户端要获取锁，就在lock节点下创建**临时顺序节点**。

- 然后获取lock下面的所有子节点，客户端获取所有的节点后，如果发现自己创建的子节点序号最小，那么就认为客户端获取到了锁。使用碗锁后，将该节点删除。
- 如果发现自己创建的子节点并非lock所有子节点中最小的，说明没有获取到锁，此时客户端需要找到比自己小的那个节点，同时对注册事件监听，删除监听事件。
- 如果发现比自己小的那个节点被删除，则客户端的Watch会收到相应的通知，此时再次判断自己创建的节点是否是lock子节点中序号最小的，如果是则获得了锁，如果不是则重复上述步骤继续获取到比自己小的一个节点并注册监听。

![image-20240526151319594](https://raw.githubusercontent.com/PeipengWang/picture/master/zk/image-20240526151319594.png)



#### 1）可重入锁Shared Reentrant Lock

Shared意味着锁是全局可见的， 客户端都可以请求锁。 Reentrant和JDK的ReentrantLock类似， 意味着同一个客户端在拥有锁的同时，可以多次获取，不会被阻塞。 它是由类InterProcessMutex来实现。 它的构造函数为：

```java
public InterProcessMutex(CuratorFramework client, String path)
```

通过acquire获得锁，并提供超时机制：

```java
/**
* Acquire the mutex - blocking until it's available. Note: the same thread can call acquire
* re-entrantly. Each call to acquire must be balanced by a call to release()
*/
public void acquire();
 
/**
* Acquire the mutex - blocks until it's available or the given time expires. Note: the same thread can
* call acquire re-entrantly. Each call to acquire that returns true must be balanced by a call to release()
* Parameters:
* time - time to wait
* unit - time unit
* Returns:
* true if the mutex was acquired, false if not
*/
public boolean acquire(long time, TimeUnit unit);

```

通过release()方法释放锁。 InterProcessMutex 实例可以重用。 Revoking ZooKeeper recipes wiki定义了可协商的撤销机制。 为了撤销mutex, 调用下面的方法：

```java
/**
* 将锁设为可撤销的. 当别的进程或线程想让你释放锁时Listener会被调用。
* Parameters:
* listener - the listener
*/
public void makeRevocable(RevocationListener<T> listener)
```

#### 2）不可重入锁Shared Lock

使用InterProcessSemaphoreMutex，调用方法类似，区别在于该锁是不可重入的，在同一个线程中不可重入

#### 3）可重入读写锁Shared Reentrant Read Write Lock

类似JDK的ReentrantReadWriteLock. 一个读写锁管理一对相关的锁。 一个负责读操作，另外一个负责写操作。 读操作在写锁没被使用时可同时由多个进程使用，而写锁使用时不允许读 (阻塞)。 此锁是可重入的。一个拥有写锁的线程可重入读锁，但是读锁却不能进入写锁。 这也意味着写锁可以降级成读锁， 比如请求写锁 —>读锁 —->释放写锁。 从读锁升级成写锁是不成的。 主要由两个类实现：

```
InterProcessReadWriteLock
InterProcessLock
```

#### 4）信号量Shared Semaphore

一个计数的信号量类似JDK的Semaphore。 JDK中Semaphore维护的一组许可(permits)，而Cubator中称之为租约(Lease)。注意，所有的实例必须使用相同的numberOfLeases值。 调用acquire会返回一个租约对象。 客户端必须在finally中close这些租约对象，否则这些租约会丢失掉。 但是， 但是，如果客户端session由于某种原因比如crash丢掉， 那么这些客户端持有的租约会自动close， 这样其它客户端可以继续使用这些租约。 租约还可以通过下面的方式返还：

```java
public void returnAll(Collection<Lease> leases)
public void returnLease(Lease lease)

```

注意一次你可以请求多个租约，如果Semaphore当前的租约不够，则请求线程会被阻塞。 同时还提供了超时的重载方法:

```java
public Lease acquire()
public Collection<Lease> acquire(int qty)
public Lease acquire(long time, TimeUnit unit)
public Collection<Lease> acquire(int qty, long time, TimeUnit unit)
```

主要类有:

```
InterProcessSemaphoreV2
Lease
SharedCountReader
```

#### 5）多锁对象Multi Shared Lock

Multi Shared Lock是一个锁的容器。 当调用acquire， 所有的锁都会被acquire，如果请求失败，所有的锁都会被release。 同样调用release时所有的锁都被release(失败被忽略)。 基本上，它就是组锁的代表，在它上面的请求释放操作都会传递给它包含的所有的锁。
主要涉及两个类：

```
InterProcessMultiLock
InterProcessLock
```

它的构造函数需要包含的锁的集合，或者一组ZooKeeper的path。

```java
public InterProcessMultiLock(List<InterProcessLock> locks)
public InterProcessMultiLock(CuratorFramework client, List<String> paths)
```

### 栅栏barrier

1）DistributedBarrier构造函数中barrierPath参数用来确定一个栅栏，只要barrierPath参数相同(路径相同)就是同一个栅栏。通常情况下栅栏的使用如下：
 1.主导client设置一个栅栏
 2.其他客户端就会调用waitOnBarrier()等待栅栏移除，程序处理线程阻塞
 3.主导client移除栅栏，其他客户端的处理程序就会同时继续运行。
 DistributedBarrier类的主要方法如下：
 setBarrier() - 设置栅栏
 waitOnBarrier() - 等待栅栏移除
 removeBarrier() - 移除栅栏

2）双栅栏Double Barrier
 双栅栏允许客户端在计算的开始和结束时同步。当足够的进程加入到双栅栏时，进程开始计算，当计算完成时，离开栅栏。双栅栏类是DistributedDoubleBarrier DistributedDoubleBarrier类实现了双栅栏的功能。它的构造函数如下：

```java
// client - the client
// barrierPath - path to use
// memberQty - the number of members in the barrier
public DistributedDoubleBarrier(CuratorFramework client, String barrierPath, int memberQty)
```

memberQty是成员数量，当enter方法被调用时，成员被阻塞，直到所有的成员都调用了enter。当leave方法被调用时，它也阻塞调用线程，直到所有的成员都调用了leave。
 注意：参数memberQty的值只是一个阈值，而不是一个限制值。当等待栅栏的数量大于或等于这个值栅栏就会打开！
 与栅栏(DistributedBarrier)一样,双栅栏的barrierPath参数也是用来确定是否是同一个栅栏的，双栅栏的使用情况如下：
 1.从多个客户端在同一个路径上创建双栅栏(DistributedDoubleBarrier),然后调用enter()方法，等待栅栏数量达到memberQty时就可以进入栅栏。
 2.栅栏数量达到memberQty，多个客户端同时停止阻塞继续运行，直到执行leave()方法，等待memberQty个数量的栅栏同时阻塞到leave()方法中。
 3.memberQty个数量的栅栏同时阻塞到leave()方法中，多个客户端的leave()方法停止阻塞，继续运行。
 DistributedDoubleBarrier类的主要方法如下： enter()、enter(long maxWait, TimeUnit unit) - 等待同时进入栅栏
 leave()、leave(long maxWait, TimeUnit unit) - 等待同时离开栅栏
 异常处理：DistributedDoubleBarrier会监控连接状态，当连接断掉时enter()和leave方法会抛出异常。

### 计数器Counters

利用ZooKeeper可以实现一个集群共享的计数器。 只要使用相同的path就可以得到最新的计数器值， 这是由ZooKeeper的一致性保证的。Curator有两个计数器， 一个是用int来计数，一个用long来计数。

#### 1）SharedCount

这个类使用int类型来计数。 主要涉及三个类。

```
* SharedCount
* SharedCountReader
* SharedCountListener
```

SharedCount代表计数器， 可以为它增加一个SharedCountListener，当计数器改变时此Listener可以监听到改变的事件，而SharedCountReader可以读取到最新的值， 包括字面值和带版本信息的值VersionedValue。

#### 2）DistributedAtomicLong

除了计数的范围比SharedCount大了之外， 它首先尝试使用乐观锁的方式设置计数器， 如果不成功(比如期间计数器已经被其它client更新了)， 它使用InterProcessMutex方式来更新计数值。 此计数器有一系列的操作：

- get(): 获取当前值
- increment()： 加一
- decrement(): 减一
- add()： 增加特定的值
- subtract(): 减去特定的值
- trySet(): 尝试设置计数值
- forceSet(): 强制设置计数值

你必须检查返回结果的succeeded()， 它代表此操作是否成功。 如果操作成功， preValue()代表操作前的值， postValue()代表操作后的值。

## End

Curator抽象和简化了很多复杂的zookeeper操作，是zk使用者的福音。而要彻底的幸福，那就是不再使用它。

我不知道其他人把zk放在一个什么位置，但在我接触paxos协议之后，就很难对它产生浓厚的兴趣。一般在技术选型的时候，它会躺在我的备选列表最后，我甚至根本无法掌握源代码里那些晦涩难懂的逻辑。

但工程建设从来不以我们的喜好来进行衡量。从来如此。

# 转载来源

作者：小姐姐味道
链接：https://juejin.cn/post/6844903632064741384
