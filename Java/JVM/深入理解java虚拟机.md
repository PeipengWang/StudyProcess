
# 深入理解java虚拟机
## Java发展历程  
标志性事件   
2012年JDK7仓促上线，很多规划的功能被砍掉  
2014年JDK8上线，陷入Jigsaw模块化功能深坑  
2017年经过艰苦的谈判，JDK9加入了Jigsaw模块化，增强了若干工具  
此后，Oracle对JDK开发进入敏捷开发阶段，每三个月和九个月发布一个新版本，由于迭代速度过快，决定每六个JDK大版本中一个才会被长期支持（LTS版）。JDK8、JDK11、JDK17为LTS版。  
2018年9月JDK11开始维护两个版本，OpenJDK和OracleJDK，两个JDK共享大部分源码，前者可以商用免费，但只有半年的更新支持，但是后者在生产环境中商用必须付费。  
2019年2月RedHat公司接受JDK8和JDK11的维护工作  
# Java 内存区域与相应内存溢出异常  
对于Java程序员来说，在虚拟机自动内存管理的机制下，不需要再为每一个new操作去写配对的delete/free代码，不容易出现内存溢出与内存泄露的问题，但是，一旦出现这种问题，如果不了解虚拟机怎样运行的，修正问题将会异常艰巨。  
## 数据区  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/87f2f7faadca4f3f9ea1bf1a7ddb531b.png)
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/ecaf80a7c38d496ba24eed9ff387c0f2.png)
可以笼统的将Java虚拟机内存分为堆（Heap）内存和栈（Stack)内存，  

### 程序计数器（线程私有）  
可以看做是当前线程执行的字节码的行号指示器。由于Java虚拟机的多线程是通过线程轮流切换、分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器只会执行一条线程的指令。因此，为了线程切换后能够恢复到正确的执行位置，每条线程都需要一个独立的程序计数器，每条线程互不影响，这类内存被称为“线程私有”的内存。  
如果线程正在执行一个Java方法，这个计数器记录的是蒸菜执行的虚拟机字节码指令的地址；如果执行的是native方法，这个计数器指针为空（Undefined）。  
### Java虚拟机栈（线程私有）  
Java虚拟机栈也是线程私有的，它的生命周期与线程相同，虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机栈都会同步创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每个方法被调用直到执行完毕。每一个方法被调用直至执行完毕的过程，对应着一个栈帧在虚拟机栈从入栈到出栈的过程。  
局部变量表存储了编译可知的基本数据类型（boolean、byte、char、short、float、long、double）、对象引用和returnAdrerss类型（指向一条字节码指令的地址）。  
对这个区域规定了两种异常情况：如果线程请求的深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；如果Java虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出OutOfMemoryError异常。  
主要异常：  
NullPointerException - 空指针引用异常  
ClassCastException - 类型强制转换异  
IllegalArgumentException - 传递非法参数异常  
ArithmeticException - 算术运算异常  
ArrayStoreException - 向数组中存放与声明类型不兼容对象异常  
IndexOutOfBoundsException - 下标越界异常  
NegativeArraySizeException - 创建一个大小为负数的数组错误异常  
NumberFormatException - 数字格式异常  
SecurityException - 安全异常  
UnsupportedOperationException - 不支持的操作异常  
### 本地方法栈（线程私有）  
本地方法栈与虚拟机栈的主要区别是，本地方法为虚拟机所用到的native方法服务，虚拟机栈为虚拟机执行Java方法。  
### 堆（线程公有）
堆（Heap）是虚拟机所管理内存中最大的一块，Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域唯一目的是存放对象实例，Java世界里“几乎”所有实例都在这里分配内存。
Java堆是垃圾收集器管理的内存区域，因此也被成为不GC堆。  
### 方法区（线程公有）
方法区（Method Area）和Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存数据。  
#### 运行时常量池
运行时常量池是方法区的一部分，用于存放编译期生成的各种字面量与符号引用，具有动态特性，Java语言并不要求常量一定在编译期间才能产生，也就是说，运行期间可以将新的常量放入池中，这种特性被开发人员利用比较多的是String类的intern()方法。  
### 方法区与堆的区别
在Java虚拟机 (JVM) 中，堆和方法区是两个不同的内存区域，它们分别用于存储不同类型的数据。  
堆 (Heap):  
用途: 主要用于存储对象实例，包括程序运行时动态创建的对象和数组。  
生命周期: 堆中的对象生命周期与其引用的作用域和生命周期相关。当没有引用指向一个对象时，它就成为垃圾，由垃圾收集器负责回收。  
特点: 堆是线程共享的区域，所有线程都共享相同的堆，但每个线程都有自己的栈。  
方法区 (Method Area):  
用途: 主要用于存储类的信息、静态变量、常量、方法代码等。在方法区中，存储的是类级别的信息，而不是实例级别的信息。  
生命周期: 随着类加载而加载，随着类卸载而卸载。在运行时，类的字节码、静态变量、常量等信息都存储在方法区。  
特点: 方法区也是线程共享的，存储的是类相关的信息。  
虽然Java 8及之前的版本中有“永久代”来实现方法区，但在Java 8后，永久代被元空间（Metaspace）取代。元空间也属于方法区的一部分，但它的实现方式和内存分配方式与永久代有很大不同。  
总体来说，堆和方法区在用途、生命周期和存储内容上有明显的区别。堆主要用于存储对象实例，而方法区主要用于存储类相关的信息。  
## HotSpot虚拟机对象（new一个对象内存会发生什么）  
### 对象的创建  
当虚拟机遇到一条字节码new指令时，首先检查这个指令的参数是否能在常量池中定位一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须执行相应的类加载过程，在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可以完全确定，为对象分配内存的任务等同于把一块确定大小的内存从Java堆中划分出来。内存分配完成后，虚拟机必须将分配的内存空间都初始化为零值。  
接下来，Java虚拟机还要对对象进行必要的设置，例如哪个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码（调用hashCode()方法计算出）、对象的分代年龄等信息。  
然后对象创建完成了，对于Java程序员来说才刚刚开始-构造函数。  
### 对象的内存布局

在HotSpot虚拟机里，对象的堆内存中存储布局可以划分为三个部分：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。  
对象头分为两部分：一部分是用于存储对象自身的运行时数据，如哈希码、GC分代年龄、锁状态标志、线程持有锁、偏向线程ID、偏向时间戳等，这部分数据可能为32bit或者64bit，被官方称为“Mark Word”；另一部分是类型指针，即对象指向它的类型元数据的指针，Java虚拟机通过这个指针来确定对象是哪个类的实例。  
实例数据是对象真正存储的有效信息，无论是从父类继承的还是在子类中定义的都必须记录起来。  
第三部分是对齐填充，仅仅起到占位符的作用。  
### 对象的访问
两种访问方式：  
句柄访问，Java堆会划分处一块内存作为句柄池，栈中的reference中存储的就是对象的句柄池，而句柄中包含了对象实例数据与类型数据各自具体的地址信息结构如下所示  
在这里插入图片描述  
直接指针访问，reference存放的是直接对象的地址，内存布局需要考虑如何存放对象。好处是速度快，HotSpot主要使用这种方式。  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/bdc515c91ca34e1dab1c65d6404e5bd2.png)  

## 虚拟机设置内存大小  
先罗列网友整理的表格：  
![](https://raw.githubusercontent.com/PeipengWang/picture/master/14792c5df30e4a97b64109762d7735eb.png)

### 堆
-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError  
其中+HeapDumpOnOutOfMemoryError参数可以是内存溢出存储内存快照，可用于分析错误。  
### 虚拟机栈和本地方法栈  
-Xss128k -XX:+HeapDumpOnOutOfMemoryError  
虚拟机栈之前说过有两种溢出，线程请求的栈深度大于虚拟机允许的深度，将抛出StackOverflowError异常。如果虚拟机栈可以动态扩展，当扩展的时候没有申请到内存的时候抛出OutOfMemoryError。
一个机器的内存是有限的。建立过多线程导致的内存溢出。如果不能减少线程，那么可以通过减少堆和减少栈容量来换取更多线程。这部分可以深入讨论，主要讨论各个模块的关系，和各自的限制。  
### 方法区
jdk7：-XX:PermSize=10m -XX:MaxPermSize=10m  
jdk8：-XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m  
jdk8的方法区变成了元空间，使用的是直接内存。所以和jdk7使用的参数不一样  
### 直接内存
直接内存如果不设置和-Xmx(堆最大)一样。可通过参数-XX:MaxDirectMemorySize来设置直接内存。直接内存一般和NIO有关，DirectByteBuffer中通过Unsafe去操作的直接内存，DirectByteBuffer是通过计算知道内存不足抛出的异常，unsafe.allocateMemory()才是去申请内存的方法。测试代码如下图：  
测试参数：-Xms20m -XX:MaxDirectMemorySize=10m  
### 设置实例--需要补充
## 垃圾收集器与内存分配策略
### 概述
研究路线：哪些内存需要回收，什么时候回收，如何回收  
程序计数器，虚拟机栈，本地方法栈三个区域随线程而生也随线程而灭，每个栈帧中分配的内存基本上在类结构确定下来时都是已知的，因此这几个区域的内存回收都具备确定性。  
Java堆和方法区有着显著的不确定性，一个接口的多个实现类所需的内存可能会不一样，一个方法所执行的不同条件分支所需要的内存也可能不一样，处于运行期间，我们才知道究竟创建哪些对象，创建多少个对象，这部分的回收是动态的。  
### 判断对象是否死去的方法
#### 回收计数法
在对象添加一个计数器，引用一次就加一，引用失效就减一，当计数器为0时就判断为死亡。  
优点：原理简单，引用效率高  
缺点：当两个对象互相引用的时候，计数器永远都不会我0，即使两个对象都已经不再使用了，这使得无法回收。  
#### 可达性分析法
从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。不可达对象。  
在Java语言中，GC Roots包括：  
虚拟机栈中引用的对象。  
方法区中类静态属性实体引用的对象。  
方法区中常量引用的对象。  
本地方法栈中JNI引用的对象。  
#### 引用
分为强引用、软引用、弱引用、虚引用四种。  
#### 生存还是死亡
即使可达性分析算法中判定为不可达对象，这时候需要进一步去判断，至少经过两次标记过程：可达性分析进行第一次标记，第一次标记后进行筛选，筛选条件是此对象是否有必要执行finalize()方法。  

### 垃圾收集算法
#### 分代收集理论
分代收集理论建立在两个假说之上：  
绝大多数对象都是朝生夕灭的；  
熬过多次垃圾收集对象的过程就越难以消亡。  
这两个分代假说奠定了多款常用的垃圾收集器的一致性设计原则：收集器将Java堆划分成不同的区域，然后将回收对象依据其年龄分配到不同的区域中存储。垃圾回收器每次只回收其中一个或者一部分的区域，因此有了“MinorGC” “MajorGC”“FullGC”这样回收类型的划分。也因此产生根据其特征相匹配的收集算法：“标记-复制算法”“标记-清除算法”“标记-整理算法”等针对垃圾收集器的算法。  
把分代收集理论具体放到现在的商用Java虚拟机中，设计者一般把Java堆分为新生代（YoungGereration）和老年代（OldGeneration）两个区域。  
#### 标记-清除算法
“标记-清除”（Mark-Sweep）算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的。  

它的主要缺点有两个：一个是效率问题，标记和清除过程的效率都不高；另外一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。  

#### 标记-复制算法
“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。  
这样使得每次都是对其中的一块进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半，持续复制长生存期的对象则导致效率降低。  

#### 标记-整理算法
复制收集算法在对象存活率较高时就要执行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。   
根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/31c91626f2fa44f6ad5d5c1a7145e0e1.png)

### 内存分配
在学习垃圾收集器的种类之前首先需要了解在堆中Eden区和Old区的对象内存分配  
#### 对象优先在Eden分配  
大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够的空间进行分配时，虚拟机将发起异常Minor GC。  
#### 大对象直接进入老年代
大对象是指需要连续庞大连续内存空间的java对象，最典型的大对象就是那种很长的字符串，或者元素数量很庞大的数据。大对象对虚拟机的分配是一个不折不扣的坏消息，尤其是短命的坏消息，写程序时应该注意。
#### 长期存活的对象进入老年代
对象在Eden去诞生，如果第一次MinorGC能够存活的话将进入survivor区（如果能容纳的话），每过一次MinorGC，年龄就加一，年龄增长到一定程度（默认15岁），对象放入老年代。
#### 动态对象年龄判定
如果Survivor空间中相同年龄的对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，不用等到设置的年龄。
#### 空间分配担保
在发生MinorGC之前虚拟机必须先检查老年代的最大可用的连续空间是否大于新生代所有对象的总空间，如果条件成立，那么这次MinorGC可用确保是安全的，如果不成立，需要查看是否设置允许担保失败，如果不允许，则执行一次FullGC，如果允许，则继续检查老年代的连续空间是否大于每次晋升到老年代对象的平均大小，如果大于，则尝试一次具有风险的MinirGC，如果小于，则进一步进行FullGC。
### 垃圾收集器种类

## 虚拟机分析工具
### 基础故障处理工具
#### jps虚拟机进程状况监视工具
罗列正在运行的虚拟机进程，并显示虚拟机执行的主类名称以及这些进程的本地虚拟机唯一id。无法定位进程名称时可以依赖jsp来区分。  
格式 jsp [options] [pid]  
例如jps、 jps -l  
选项：  
-q：只输出LVMID，省略主类名称  
-m：输出虚拟机启动时传递给主类main()函数的参数  
-l: 输出主类全名，如果进程执行的是jar包，则输出jar路径  
-v：输出虚拟机进程启动时的JVM参数  
#### jstat虚拟机统计信息监视工具
用于监视虚拟机各种运行状态信息的命令行工具。它可以显示进程中的类加载、内存、垃圾收集信息、即时编译等运行的数据。  
命令格式：  
jstat [option vmid [interval [s|ms]  [count] ] ]  
interval和count代表查询间隔和次数，如果省略这两个参数，说明只会查询一次。  
例如：jstat -gc 24408 1000 100  
1秒钟查询一次堆情况，总共查询100次  
选项option代表用户查询虚拟机信息，主要分为三类：类加载、垃圾收集、运行是编译情况  
-class  显示有关类加载器行为的统计信息  
-compiler 显示有关 Java HotSpot 虚拟机即时编译器行为的统计信息  
-gc 显示垃圾收集堆行为的统计信息  
-gccapacity 显示各代容量及其相应空间的统计信息  
-gccause 显示垃圾收集统计信息（与 -gcutil 相同）的摘要，以及上次和当前（如果适用）垃圾收集事件的原因  
-gcnew 显示新一代行为的统计数据  
-gcnewcapacity 显示关于新一代大小及其相应空间的统计信息  
-gcold 显示老一代行为的统计信息和元空间统计信息  
-gcoldcapacity 显示关于老一代大小的统计数据  
-gcmetacapacity 显示关于元空间大小的统计信息  
-gcutil 显示垃圾收集统计信息的摘要  
 jstat -gcutil 24408  
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT     
  0.00   0.00  13.01   4.91  97.62  91.53      2    0.035     1    0.051    0.086  
 S0,S1 代表Survivor区，E代表Eden区，O代表老年代，P代表永久代，程序运行以来总共发生YoungGC 2次，总耗时0.035s，FullGC 1次，总耗时0.086s。  
 查看10次GC信息    
 jstat -gcutil 39128 1000 10    
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
 74.02   0.00  93.88  25.82  71.07  61.60   1112    3.283     0    0.000    3.283
 74.10   0.00  20.37  25.82  71.07  61.60   1116    3.289     0    0.000    3.289
 74.19   0.00  36.18  26.39  71.07  61.60   1120    3.294     0    0.000    3.294
 74.27   0.00  33.59  26.39  71.07  61.60   1124    3.300     0    0.000    3.300
  6.25   0.00  12.52  26.95  71.07  61.60   1128    3.308     0    0.000    3.308
  0.00  74.44  33.82  26.95  71.07  61.60   1131    3.343     0    0.000    3.343
  0.00  74.52  35.58  26.95  71.07  61.60   1133    3.345     0    0.000    3.345
 74.57   0.00  86.44  26.95  71.07  61.60   1134    3.346     0    0.000    3.346
 74.67   0.00  46.28  26.95  71.07  61.60   1136    3.349     0    0.000    3.349
 74.77   0.00  10.35  26.95  71.07  61.60   1138    3.352     0    0.000    3.352
通过检查十次的gc发现 :    新生代的数据增长到数据满时会触发异常YGC，同时将数据放到老年代  
#### jinfo:Java配置信息工具  
查看JVM配置信息  
命令格式：  
jinfo [option] pid  
例如  
[10307006@zte.intra@LIN-CCE6AF6663D ~]$ jinfo -flag InitialHeapSize 35487  
-XX:InitialHeapSize=197132288  
#### jmap:Java内存映像工具  
jmap 用于生成堆转储快照，还可以查询finalize执行队列、Java堆和方法区的详细信息，如空间使用率、当前用的哪种平台收集器。  
命令格式：  
jmap [options] vmid  
其中options的选线可以为：  
-dump 生成堆转快照的文件。格式为-dump:[live,]format=b,file=filename，其中live子参数说明是否只dump出存活对象  
-finalizerinfo 显示在F-Queue中等待FinaLizer线程执行finalize方法的对象。  
-heap 显示堆详细信息。 例如：jmap -heap 35487 简单罗列部分数据如下  
```
jmap -heap 35487
...
JVM version is 25.242-b08  
using thread-local object allocation.  
垃圾回收算法  
Parallel GC with 6 thread(s)  
参数配置  
Heap Configuration:  
   MinHeapFreeRatio         = 0  
   MaxHeapFreeRatio         = 100  
   ...
分代情况  
Heap Usage:  
PS Young Generation  
Eden Space:  
   capacity = 99614720 (95.0MB)  
   used     = 9789096 (9.335609436035156MB)  
   free     = 89825624 (85.66439056396484MB)  
   9.826957301089639% used  
....
```
-histo 显示堆中对象统计信息，包括类、实例对象、合计容量。  
-permastat 以ClassLoader为统计口径显示永久代内存信息。  
#### jhat：虚拟机堆转储快照分析工具  
JDK提供jhat命令与jmap搭配使用，来分析jmap生成的堆转储快照。  
例如：  
jhat ommpjmap.bin   
Reading from ommpjmap.bin...  
Dump file created Sat Feb 19 10:48:30 CST 2022  
Snapshot read, resolving...  
Resolving 110871 objects...  
Chasing references, expect 22 dots......................  
Eliminating duplicate references......................  
Snapshot resolved.  
Started HTTP server on port 7000  
Server is ready.  
在浏览器输入localhost:7000即可看到分析的堆信息  
#### jstack:Java堆栈跟踪工具  
jstack命令用于生成虚拟机当前时刻的线程快照。线程快照就是当前虚拟机每一条线程正在执行的方法堆栈的集合。  
命令格式：  
jstack [options] vmid  
options选项合法值和具体含义如表所示  
-F 当正常输出的请求不被响应时，强制输出线程堆栈。  
-l 除堆栈外，显示关于锁的附加信息  
-m 如果调用本地方法的话，可以输出C/C++的堆栈  
### 可视化处理工具  
JVM集成的可视化工具包括JConsole、JHSDB、VisualVM和JMC四个。  
### 调优案例

# 虚拟机执行子系统
代码编译从本地机器码转变为字节码  
## 类文件结构
Java虚拟机不包括Java语言在内的任何程序语言绑定，它只与“Class文件”这种特定的二进制文件格式所关联。Java技术能够一直保持良好的向后兼容性，Class文件结构的稳定功不可没。主要将如何将Class文件存储格式的具体细节。  
### 常量池
常量池主要存放两大类常量：字面量和符号引用。字面量比较接近Java语言层面的常量概念，如文本字符串、被声明的final常量值等。而符号引用则属于编译原理方面的概念。  
### 操作码
## 虚拟机类加载机制
虚拟机加载机制实际为Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，并最终形成可以被虚拟机直接使用的Java类型的过程。  
### 类加载的过程
一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期将会经历加载、验证、准备、解析、初始化、使用、卸载七个阶段，其中验证、准备、解析被统称为连接。  
#### 加载  
在加载阶段，虚拟机会完成三件事：  
通过一个类的全限定性类名来获取此类的二进制字节流。  
通过这个字节流所代表的静态存储结构转化为运行时数据结构。  
在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据访问入口。  
#### 验证
目的是确保Class文件的字节流包含的信息不会危害虚拟机自身的安全。    
#### 准备
准备阶段是正式为类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置变量的初始值阶段，从概念上讲，这些变量所使用的内存都应当在方法区中进行分配， 方法区实际是是一个逻辑区域，JDK8之后类变量会随着Class对象一起放到Java堆中。    
需要注意的是类变量会在准备阶段进行内存分配，但是实例变量会在对对象进行实例化的时候随对象一起放到Java堆中。    
在准备阶段会对类变量进行初始化，如果这个类变量用final修饰，则会初始化为设置的值，如果不是final类型的则设置为0，false或者null。    
#### 解析
解析阶段是Java虚拟机将常量池内的符号引用替换为直接引用的过程。    
解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符这七类符号引用今行，分别对应常量池的八种常量类型的表格。    
#### 初始化
之前的阶段除了用户应用程序可以通过自定义加载器的方式局部参与外，其余动作都完全由Java虚拟机来主导控制，直到初始化阶段，Java虚拟机才真正开始执行类中编写的程序代码，将主导权交给应用程序。    
### 类加载器
Java虚拟机设计团队有意把“通过一个全限定名来获取描述改类的二进制字节流”这个动作放到Java虚拟机外部，以便让应用程序自己决定如何获取所需要的类。实现这个动作的代码被称为“类加载器”。
站在Java虚拟机的角度看，只存在两种类加载器，一种是启动类加载器，在JVM内部，另一种是其它所有类加载器，独立于虚拟机外面，并且全部继承自抽象类java.lang.ClassLoader    
#### 双亲委派模型
双亲委派模型的工作流程是：    
如果一个类加载器受到了类加载的请求，它不会自己尝试加载这个类，而是委托它的父类加载器去完成，每一个层次的类加载器都是这样，因此都传输到顶层的启动类加载器中，只有当父类加载器反馈无法完成这个加载请求的时候（搜索范围内没有需要的类），子类加载器才会去完成加载。    
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/e242319ab7bf4996890383ae527d87e8.png)  

1、启动类加载器(Bootstrap ClassLoader),它是属于虚拟机自身的一部分，用C++实现的，主要负责加载<JAVA_HOME>\lib目录中或被-Xbootclasspath指定的路径中的并且文件名是被虚拟机识别的文件。它等于是所有类加载器的爸爸。    
2、扩展类加载器(Extension ClassLoader),它是Java实现的，独立于虚拟机，主要负责加载<JAVA_HOME>\lib\ext目录中或被java.ext.dirs系统变量所指定的路径的类库。    
3、应用程序类加载器(Application ClassLoader),它是Java实现的，独立于虚拟机。主要负责加载用户类路径(classPath)上的类库，如果我们没有实现自定义的类加载器那这玩意就是我们程序中的默认加载器。    
```
       {
            // First, check if the class has already been loaded 先判断class是否已经被加载过了
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);  //找他爸爸去加载
                    } else {
                        c = findBootstrapClassOrNull(name);  //没爸爸说明是顶层了就用Bootstrap ClassLoader去加载
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);    //最后如果没找到，那就自己找

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
```
####  为什么需要破坏双亲委派？
因为在某些情况下父类加载器需要委托子类加载器去加载class文件。受到加载范围的限制，父类加载器无法加载到需要的文件，以Driver接口为例，由于Driver接口定义在jdk当中的，而其实现由各个数据库的服务商来提供，比如mysql的就写了MySQL Connector，那么问题就来了，DriverManager（也由jdk提供）要加载各个实现了Driver接口的实现类，然后进行管理，但是DriverManager由启动类加载器加载，只能记载JAVA_HOME的lib下文件，而其实现是由服务商提供的，由系统类加载器加载，这个时候就需要启动类加载器来委托子类来加载Driver实现，从而破坏了双亲委派，这里仅仅是举了破坏双亲委派的其中一个情况。    

## 高效并发
### Java内存模型与线程  
计算机的运算速度和它的存储和通信子系统的速度差距太大，大量的时间都花在磁盘I/O、通信网络或者数据库访问上。如果不希望处理器在大部分时间都在等待其它资源的空闲状态，就必须使用一些时间把处理器的运算能力压榨出来。    
衡量一个服务器性能的高低好坏，每秒事务处理数（TPS）是重要指标之一。而TPS与程序的并发能力密切相关    




























# 问题：
## 局部变量表中什么是对象引用和returnAdress  
## native方法与java方法的区别  
## String类的intern()方法有什么用？  
String::intern()是一个本地方法，它的作用是如果字符串常量池已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象引用；否则，会将此String对象包含的字符串添加到常量池中，并且返回此String对象引用。  
## finalize()方法作用  
由上，Java中的对象并不一定会被全部垃圾回收，当你不想要该对象的时候，你需要手动去处理那些“特殊内存”。  
Java 允许定义这样的方法，它在对象被垃圾收集器析构(回收)之前调用，这个方法叫做 finalize( )，它用来清除回收对象。  
不建议用finalize()方法完成“非内存资源”的清理工作，但建议用于：  
① 清理本地对象(通过JNI创建的对象)；  
② 确保某些非内存资源(如Socket、文件等)的释放：在finalize()方法中显式调用其他资源释放方法。  
finalize()的隐藏问题  
System.gc()与System.runFinalization()方法增加了finalize()方法执行的机会，但不可盲目依赖它们  
Java语言规范并不保证finalize方法会被及时地执行、而且根本不会保证它们会被执行  
finalize()方法可能会带来性能问题。因为JVM通常在单独的低优先级线程中完成finalize()的执行  
对象再生问题：finalize()方法中，可将待回收对象赋值给GC Roots可达的对象引用，从而达到对象再生的目的  
finalize()方法至多由GC执行一次（用户当然可以手动调用对象的finalize()方法，但并不影响GC对finalize()的行为）  
finalize()一般不用！被执行的不确定性太大。不要指望使用finalize()来回收你的对象，它只会在系统进行GC的时候清理特殊内存，不受你的控制！  
finalize()的执行过程(生命周期)  
当对象变成(GC Roots)不可达时，GC会判断该对象是否覆盖了finalize方法，若未覆盖，则直接将其回收。否则，若对象未执行过finalize方法，将其放入F-Queue队列，由一低优先级线程执行该队列中对象的finalize方法。执行finalize方法完毕后，GC会再次判断该对象是否可达，若不可达，则进行回收，否则，对象“复活”  

## 常量池存储的数据
final修饰的变量  
。。。。  
Class文件的全路径类名  
##  类变量和实例变量的区别  
类变量：    
类变量也称为静态变量，在类中以static关键字声明，但必须在方法构造方法和语句块之外。    
无论一个类创建了多少个对象，类只拥有类变量的一份拷贝。    
静态变量除了被声明为常量外很少使用。常量是指声明为public/private，final和static类型的变量。常量初始化后不可改变。    
静态变量储存在静态存储区。经常被声明为常量，很少单独使用static声明变量。    
静态变量在程序开始时创建，在程序结束时销毁。    
与实例变量具有相似的可见性。但为了对类的使用者可见，大多数静态变量声明为public类型。    
默认值和实例变量相似。数值型变量默认值是0，布尔型默认值是false，引用类型默认值是null。变量的值可以在声明的时候指定，也可以在构造方法中指定。此外，静态变量还可以在静态语句块中初始化。    
静态变量可以通过：ClassName.VariableName的方式访问。    
类变量被声明为public static final类型时，类变量名称必须使用大写字母。如果静态变量不是public和final类型，其命名方式与实例变量以及局部变量的命名方式一致。    
实例变量：    
实例变量声明在一个类中，但在方法、构造方法和语句块之外；  
当一个对象被实例化之后，每个实例变量的值就跟着确定；  
实例变量在对象创建的时候创建，在对象被销毁的时候销毁；  
实例变量的值应该至少被一个方法、构造方法或者语句块引用，使得外部能够通过这些方式获取实例变量信息；  
实例变量对于类中的方法、构造方法或者语句块是可见的。一般情况下应该把实例变量设为私有。通过使用访问修饰符可以使实例变量对子类可见；  
实例变量具有默认值。数值型变量的默认值是0，布尔型变量的默认值是false，引用类型变量的默认值是null。变量的值可以在声明时指定，也可以在构造方法中指定；  
实例变量可以直接通过变量名访问。但在静态方法以及其他类中，就应该使用完全限定名：ObejectReference.VariableName。   
