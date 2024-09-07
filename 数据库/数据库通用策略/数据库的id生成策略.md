系统唯一ID是我们在设计一个系统的时候常常会遇见的问题，也常常为这个问题而纠结。生成ID的方法有很多，适应不同的场景、需求以及性能要求。所以有些比较复杂的系统会有多个ID生成的策略。下面就介绍一些常见的ID生成策略。

# 数据库自增长 序列或字段

最常见的方式。利用数据库，全数据库唯一。在MySQL中常用。

## 优点：

简单，代码方便，性能可以接受。
数字ID天然排序，对分页或者需要排序的结果很有帮助。

## 缺点：

不同数据库语法和实现不同，[数据库迁移](https://so.csdn.net/so/search?q=数据库迁移&spm=1001.2101.3001.7020)的时候或多数据库版本支持的时候需要处理。
在单个数据库或读写分离或一主多从的情况下，只有一个主库可以生成。有单点故障的风险。
在性能达不到要求的情况下，比较难于扩展。
如果遇见多个系统需要合并或者涉及到[数据迁移](https://so.csdn.net/so/search?q=数据迁移&spm=1001.2101.3001.7020)会相当痛苦。
分表分库的时候会有麻烦。

## 优化方案：

针对主库单点，如果有多个Master库，则每个Master库设置的起始数字不一样，步长一样，可以是Master的个数。比如：Master1 生成的是 1，4，7，10，Master2生成的是2, 5, 8, 11 Master3生成的是 3, 6, 9, 12。这样就可以有效生成集群中的唯一ID，也可以大大降低ID生成数据库操作的负载。

# UUID

常见的方式。可以利用数据库也可以利用程序生成，一般来说全球唯一。

## 优点：

简单，代码方便。

生成ID性能非常好，基本不会有性能问题。

全球唯一，在遇见数据迁移，系统数据合并，或者数据库变更等情况下，可以从容应对。

## 缺点：

没有排序，无法保证趋势递增。

UUID往往是使用字符串存储，查询的效率比较低。

存储空间比较大，如果是海量数据库，就需要考虑存储量的问题。

传输数据量大。

不可读。

# Redis生成ID

当使用数据库来生成ID性能不够要求的时候，我们可以尝试使用Redis来生成ID。这主要依赖于Redis是单线程的，所以也可以用生成全局唯一的ID。可以用Redis的[原子操作](https://so.csdn.net/so/search?q=原子操作&spm=1001.2101.3001.7020) INCR和INCRBY来实现。

可以使用Redis集群来获取更高的吞吐量。假如一个集群中有5台Redis。可以初始化每台Redis的值分别是1,2,3,4,5，然后步长都是5。各个Redis生成的ID为：

A：1, 6, 11, 16, 21

B：2, 7, 12, 17, 22

C：3, 8, 13, 18, 23

D：4, 9, 14, 19, 24

E：5, 10, 15, 20, 25

这个，随便负载到哪个机确定好，未来很难做修改。但是3-5台服务器基本能够满足器上，都可以获得不同的ID。但是步长和初始值一定需要事先需要了。使用Redis集群也可以方式单点故障的问题。

另外，比较适合使用Redis来生成每天从0开始的流水号。比如订单号=日期+当日自增长号。可以每天在Redis中生成一个Key，使用INCR进行累加。

# Twitter的snowflake算法

snowflake是Twitter开源的分布式ID生成算法，结果是一个long型的ID。其核心思想是：使用41bit作为毫秒数，10bit作为机器的ID（5个bit是数据中心，5个bit的机器ID），12bit作为毫秒内的流水号（意味着每个节点在每毫秒可以产生 4096 个 ID），最后还有一个符号位，永远是0。
 Twitter的 Snowflake　JAVA实现方案
核心代码为其IdWorker这个类实现，其原理结构如下，我分别用一个0表示一位，用—分割开部分的作用：

```
 1||0---0000000000 0000000000 0000000000 0000000000 0 --- 00000 ---00000 ---000000000000

 64位ID (42(毫秒)+5(机器ID)+5(业务编码)+12(重复累加))
```

 在上面的字符串中，第一位为未使用（实际上也可作为long的符号位），接下来的41位为毫秒级时间，
 然后5位datacenter标识位，5位机器ID（并不算标识符，实际是为线程标识），然后12位该毫秒内的当前毫秒内的计数，加起来刚好64位，为一个Long型。
 这样的好处是，整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞（由datacenter和机器ID作区分），并且效率较高，经测试，snowflake每秒能够产生26万ID左右，完全满足需要。

```
public class IdWorker {
  // 时间起始标记点，作为基准，一般取系统的最近时间（一旦确定不能变动）
  private final static long twepoch = 1288834974657L;
  // 机器标识位数
  private final static long workerIdBits = 5L;
  // 数据中心标识位数
  private final static long datacenterIdBits = 5L;
  // 机器ID最大值
  private final static long maxWorkerId = -1L ^ (-1L << workerIdBits);
  // 数据中心ID最大值
  private final static long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
  // 毫秒内自增位
  private final static long sequenceBits = 12L;
  // 机器ID偏左移12位
  private final static long workerIdShift = sequenceBits;
  // 数据中心ID左移17位
  private final static long datacenterIdShift = sequenceBits + workerIdBits;
  // 时间毫秒左移22位
  private final static long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

  private final static long sequenceMask = -1L ^ (-1L << sequenceBits);
  /* 上次生产id时间戳 */
  private static long lastTimestamp = -1L;
  // 0，并发控制
  private long sequence = 0L;

  private final long workerId;
  // 数据标识id部分
  private final long datacenterId;

  public IdWorker(){
    this.datacenterId = getDatacenterId(maxDatacenterId);
    this.workerId = getMaxWorkerId(datacenterId, maxWorkerId);
  }
  public IdWorker(long workerId, long datacenterId) {
    if (workerId > maxWorkerId || workerId < 0) {
      throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
    }
    if (datacenterId > maxDatacenterId || datacenterId < 0) {
      throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
    }
    this.workerId = workerId;
    this.datacenterId = datacenterId;
  }
```

```
  
   \\获取下一个ID

  public synchronized long nextId() {
    long timestamp = timeGen();
    if (timestamp < lastTimestamp) {
      throw new RuntimeException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
    

    if (lastTimestamp == timestamp) {
      // 当前毫秒内，则+1
      sequence = (sequence + 1) & sequenceMask;
     if (sequence == 0) {
        // 当前毫秒内计数满了，则等待下一秒
        timestamp = tilNextMillis(lastTimestamp);
     }
    } else {
     sequence = 0L;
   }
   lastTimestamp = timestamp;
   // ID偏移组合生成最终的ID，并返回ID
   long nextId = ((timestamp - twepoch) << timestampLeftShift)
       | (datacenterId << datacenterIdShift)
       | (workerId << workerIdShift) | sequence;

   return nextId;
  }

  private long tilNextMillis(final long lastTimestamp) {
    long timestamp = this.timeGen();
    while (timestamp <= lastTimestamp) {
      timestamp = this.timeGen();
    }
    return timestamp;
  }

  private long timeGen() {
    return System.currentTimeMillis();
  }
```


   \\获取 maxWorkerId

```
 protected static long getDatacenterId(long maxDatacenterId) {
    long id = 0L;
    try {
      InetAddress ip = InetAddress.getLocalHost();
      NetworkInterface network = NetworkInterface.getByInetAddress(ip);
      if (network == null) {
        id = 1L;
      } else {
        byte[] mac = network.getHardwareAddress();
        id = ((0x000000FF & (long) mac[mac.length - 1])
            | (0x0000FF00 & (((long) mac[mac.length - 2]) << 8))) >> 6;
        id = id % (maxDatacenterId + 1);
      }
    } catch (Exception e) {
      System.out.println(" getDatacenterId: " + e.getMessage());
    }
    return id;
  }
}
```


snowflake算法可以根据自身项目的需要进行一定的修改。比如估算未来的数据中心个数，每个数据中心的机器数以及统一毫秒可以能的并发数来调整在算法中所需要的bit数。

## 优点：

不依赖于数据库，灵活方便，且性能优于数据库。

ID按照时间在单机上是递增的。

## 缺点：

在单机上是递增的，但是由于涉及到分布式环境，每台机器上的时钟不可能完全同步，也许有时候也会出现不是全局递增的情况。

