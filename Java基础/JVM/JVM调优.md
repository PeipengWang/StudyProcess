# JVM调优
## 1、开始
JVM调优不是常规手段，性能问题一般第一选择是优化程序，最后的选择才是进行JVM调优。  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/f051959705be48668a008f321c367e6e.png)

JVM的自动内存管理本来就是为了将开发人员从内存管理的泥潭里拉出来。即使不得不进行JVM调优，也绝对不能拍脑门就去调整参数，一定要全面监控，详细分析性能数据。  

## 2、JVM调优的时机
不得不考虑进行JVM调优的是那些情况呢？  
Heap内存（老年代）持续上涨达到设置的最大内存值；  
Full GC 次数频繁；  
GC 停顿时间过长（超过1秒）；  
应用出现OutOfMemory 等内存异常；  
应用中有使用本地缓存且占用大量内存空间；  
系统吞吐量与响应性能不高或下降。  
## 3、JVM调优的目标
吞吐量、延迟、内存占用三者类似CAP，构成了一个不可能三角，只能选择其中两个进行调优，不可三者兼得。  
延迟：GC低停顿和GC低频率；  
低内存占用；  
高吞吐量;  
选择了其中两个，必然会会以牺牲另一个为代价。  
下面展示了一些JVM调优的量化目标参考实例：  
Heap 内存使用率 <= 70%;  
Old generation内存使用率<= 70%;  
avgpause <= 1秒;  
Full gc 次数0 或 avg pause interval >= 24小时 ;  
注意：不同应用的JVM调优量化目标是不一样的。  
## 4、JVM调优的步骤  
一般情况下，JVM调优可通过以下步骤进行：  
分析系统系统运行情况：分析GC日志及dump文件，判断是否需要优化，确定瓶颈问题点；  
确定JVM调优量化目标；  
确定JVM调优参数（根据历史JVM参数来调整）；  
依次确定调优内存、延迟、吞吐量等指标；  
对比观察调优前后的差异；  
不断的分析和调整，直到找到合适的JVM参数配置；  
找到最合适的参数，将这些参数应用到所有服务器，并进行后续跟踪。  
以上操作步骤中，某些步骤是需要多次不断迭代完成的。一般是从满足程序的内存使用需求开始的，之后是时间延迟的要求，最后才是吞吐量的要求，要基于这个步骤来不断优化，每一个步骤都是进行下一步的基础，不可逆行。  

## 5、常见的内存溢出
常见的 Java 内存溢出有以下三种  
（1） java.lang.OutOfMemoryError: Java heap space —-JVM Heap（堆）溢出  
JVM 在启动的时候会自动设置 JVM Heap 的值，其初始空间（即-Xms）是物理内存的1/64，最大空间（-Xmx）不可超过物理内存。可以利用 JVM提供的 -Xmn -Xms -Xmx 等选项可进行设置。Heap 的大小是 Young Generation 和 Tenured Generaion 之和。在 JVM 中如果 98％ 的时间是用于 GC，且可用的 Heap size 不足 2％ 的时候将抛出此异常信息。  
解决方法：手动设置 JVM Heap（堆）的大小。   
（2） java.lang.OutOfMemoryError: PermGen space  —- PermGen space溢出。  
PermGen space 的全称是 Permanent Generation space，是指内存的永久保存区域。为什么会内存溢出，这是由于这块内存主要是被 JVM 存放Class 和 Meta 信息的，Class 在被 Load 的时候被放入 PermGen space 区域，它和存放 Instance 的 Heap 区域不同，sun 的 GC 不会在主程序运行期对 PermGen space 进行清理，所以如果你的 APP 会载入很多 CLASS 的话，就很可能出现 PermGen space 溢出。  
解决方法： 手动设置 MaxPermSize 大小  
（3） java.lang.StackOverflowError   —- 栈溢出  
栈溢出了，JVM 依然是采用栈式的虚拟机，这个和 C 与 Pascal 都是一样的。函数的调用过程都体现在堆栈和退栈上了。调用构造函数的 “层”太多了，以致于把栈区溢出了。通常来讲，一般栈区远远小于堆区的，因为函数调用过程往往不会多于上千层，而即便每个函数调用需要 1K 的空间（这个大约相当于在一个 C 函数内声明了 256 个 int 类型的变量），那么栈区也不过是需要 1MB 的空间。通常栈的大小是 1－2MB 的。  
通常递归也不要递归的层次过多，很容易溢出

## 6、基本设置参数与几个典型配置
常见配置汇总  
堆设置  
-Xms:初始堆大小  
-Xmx:最大堆大小  
-XX:NewSize=n:设置年轻代大小  
-XX:NewRatio=n:设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
-XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5  
-XX:MaxPermSize=n:设置持久代大小  
收集器设置  
-XX:+UseSerialGC:设置串行收集器  
-XX:+UseParallelGC:设置并行收集器  
-XX:+UseParalledlOldGC:设置并行年老代收集器  
-XX:+UseConcMarkSweepGC:设置并发收集器  
垃圾回收统计信息  
-XX:+PrintGC  
-XX:+PrintGCDetails   
-XX:+PrintGCTimeStamps  
-Xloggc:filename  
并行收集器设置  
-XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数。并行收集线程数。  
-XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间  
-XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)  
并发收集器设置  
-XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况。  
-XX:ParallelGCThreads=n:设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。  
调优总结年轻代大小选择  
响应时间优先的应用：尽可能设大，直到接近系统的最低响应时间限制（根据实际情况选择）。在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象。  
吞吐量优先的应用：尽可能的设置大，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用。  
典型设置  
```
java -Xmx3550m -Xms3550m -Xmn2g –Xss128k
-Xmx3550m：设置JVM最大可用内存为3550M。
-Xms3550m：设置JVM促使内存为3550m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
-Xmn2g：设置年轻代大小为2G。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
-Xss128k：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右
```
```
java -Xmx3550m -Xms3550m -Xss128k -XX:NewRatio=4
-XX:SurvivorRatio=4
-XX:MaxPermSize=16m
-XX:MaxTenuringThreshold=0
-XX:NewRatio=4:设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5
-XX:SurvivorRatio=4：设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6
-XX:MaxPermSize=16m:设置持久代大小为16m。
-XX:MaxTenuringThreshold=0：设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增
```
吞吐量优先的并行收集器典型设置  
```
java -Xmx3800m -Xms3800m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20
-XX:+UseParallelGC：选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
-XX:ParallelGCThreads=20：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。

java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC
-XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。

java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100
-XX:MaxGCPauseMillis=100:设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。

java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
-XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
```
响应时间优先的并发收集器
```
java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
-XX:+UseConcMarkSweepGC：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
-XX:+UseParNewGC:设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
-XX:CMSFullGCsBeforeCompaction：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
-XX:+UseCMSCompactAtFullCollection：打开对年老代的压缩。可能会影响性能，但是可以消除碎片
```
打印配置
```
JVM提供了大量命令行参数，打印信息，供调试使用。主要有以下一些：
-XX:+PrintGC：输出形式：[GC 118250K->113543K(130112K), 0.0094143 secs] [Full GC 121376K->10414K(130112K), 0.0650971 secs]
-XX:+PrintGCDetails：输出形式：[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs] [GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]
-XX:+PrintGCTimeStamps -XX:+PrintGC：PrintGCTimeStamps可与上面两个混合使用
输出形式：11.851: [GC 98328K->93620K(130112K), 0.0082960 secs]
-XX:+PrintGCApplicationConcurrentTime：打印每次垃圾回收前，程序未中断的执行时间。可与上面混合使用。输出形式：Application time: 0.5291524


seconds
-XX:+PrintGCApplicationStoppedTime：打印垃圾回收期间程序暂停的时间。可与上面混合使用。输出形式：Total time for which application threads were stopped: 0.0468229 seconds
-XX:PrintHeapAtGC: 打印GC前后的详细堆栈信息。
-Xloggc:filename:与上面几个配合使用，把相关日志信息记录到文件以便分析。
```
## 7、JVM调优工具
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/ffc9253fc9f64e2d8955d852570c3019.png)

Linux 命令行工具  
进行性能监控和问题排查的时候，常常是结合操作系统本身的命令行工具来进行。  
top  
实时显示正在执行进程的 CPU 使用率、内存使用率以及系统负载等信息  
vmstat  
对操作系统的虚拟内存、进程、CPU活动进行监控  
pidstat  
监控指定进程的上下文切换  
iostat  
监控磁盘IO  
其它还有一些第三方的监控工具，同样是性能分析和故障排查的利器，如MAT、GChisto、JProfiler、arthas。  
## 8、JVM调优策略
调整内存大小  
现象：垃圾收集频率非常频繁。  
原因：如果内存太小，就会导致频繁的需要进行垃圾收集才能释放出足够的空间来创建新的对象，所以增加堆内存大小的效果是非常显而易见的。  
注意：如果垃圾收集次数非常频繁，但是每次能回收的对象非常少，那么这个时候并非内存太小，而可能是内存泄露导致对象无法回收，从而造成频繁GC。  
参数配置：  
```
 //设置堆初始值
 指令1：-Xms2g
 指令2：-XX:InitialHeapSize=2048m
 //设置堆区最大值
 指令1：`-Xmx2g` 
 指令2： -XX:MaxHeapSize=2048m
 //新生代内存配置
 指令1：-Xmn512m
 指令2：-XX:MaxNewSize=512m
```
设置符合预期的停顿时间  
现象：程序间接性的卡顿  
原因：如果没有确切的停顿时间设定，垃圾收集器以吞吐量为主，那么垃圾收集时间就会不稳定。  
注意：不要设置不切实际的停顿时间，单次时间越短也意味着需要更多的GC次数才能回收完原有数量的垃圾.  
参数配置：  
```
 //GC停顿时间，垃圾收集器会尝试用各种手段达到这个时间
 -XX:MaxGCPauseMillis 
```
调整内存区域大小比率  
现象：某一个区域的GC频繁，其他都正常。  
原因：如果对应区域空间不足，导致需要频繁GC来释放空间，在JVM堆内存无法增加的情况下，可以调整对应区域的大小比率。  
注意：也许并非空间不足，而是因为内存泄造成内存无法回收。从而导致GC频繁。  
参数配置：  
```
//survivor区和Eden区大小比率
 指令：-XX:SurvivorRatio=6  //S区和Eden区占新生代比率为1:6,两个S区2:6
//新生代和老年代的占比
 -XX:NewRatio=4  //表示新生代:老年代 = 1:4 即老年代占整个堆的4/5；默认值=2

```
调整对象升老年代的年龄  
现象：老年代频繁GC，每次回收的对象很多。  
原因：如果升代年龄小，新生代的对象很快就进入老年代了，导致老年代对象变多，而这些对象其实在随后的很短时间内就可以回收，这时候可以调整对象的升级代年龄，让对象不那么容易进入老年代解决老年代空间不足频繁GC问题。  
注意：增加了年龄之后，这些对象在新生代的时间会变长可能导致新生代的GC频率增加，并且频繁复制这些对象新生的GC时间也可能变长。  
配置参数：  
```
//进入老年代最小的GC年龄,年轻代对象转换为老年代对象最小年龄值，默认值7
 -XX:InitialTenuringThreshol=7

```
调整大对象的标准  
现象：老年代频繁GC，每次回收的对象很多,而且单个对象的体积都比较大。  
原因：如果大量的大对象直接分配到老年代，导致老年代容易被填满而造成频繁GC，可设置对象直接进入老年代的标准。  
注意：这些大对象进入新生代后可能会使新生代的GC频率和时间增加。  
配置参数：  
```
 //新生代可容纳的最大对象,大于则直接会分配到老年代，0代表没有限制。
  -XX:PretenureSizeThreshold=1000000
```
调整GC的触发时机  
现象：CMS，G1 经常 Full GC，程序卡顿严重。  
原因：G1和CMS  部分GC阶段是并发进行的，业务线程和垃圾收集线程一起工作，也就说明垃圾收集的过程中业务线程会生成新的对象，所以在GC的时候需要预留一部分内存空间来容纳新产生的对象，如果这个时候内存空间不足以容纳新产生的对象，那么JVM就会停止并发收集暂停所有业务线程（STW）来保证垃圾收集的正常运行。这个时候可以调整GC触发的时机（比如在老年代占用60%就触发GC），这样就可以预留足够的空间来让业务线程创建的对象有足够的空间分配。  
注意：提早触发GC会增加老年代GC的频率。  
配置参数：  
```
 //使用多少比例的老年代后开始CMS收集，默认是68%，如果频繁发生SerialOld卡顿，应该调小
 -XX:CMSInitiatingOccupancyFraction
 //G1混合垃圾回收周期中要包括的旧区域设置占用率阈值。默认占用率为 65%
 -XX:G1MixedGCLiveThresholdPercent=65 
```
调整 JVM本地内存大小  
现象：GC的次数、时间和回收的对象都正常，堆内存空间充足，但是报OOM  
原因： JVM除了堆内存之外还有一块堆外内存，这片内存也叫本地内存，可是这块内存区域不足了并不会主动触发GC，只有在堆内存区域触发的时候顺带会把本地内存回收了，而一旦本地内存分配不足就会直接报OOM异常。  
注意： 本地内存异常的时候除了上面的现象之外，异常信息可能是OutOfMemoryError：Direct buffer memory。 解决方式除了调整本地内存大小之外，也可以在出现此异常时进行捕获，手动触发GC（System.gc()）。  
配置参数：  
```
 XX:MaxDirectMemorySize
```
参考文献  
https://www.cnblogs.com/baihuitestsoftware/articles/6483690.html
https://cloud.tencent.com/developer/article/1812722
https://www.51cto.com/article/311739.html

