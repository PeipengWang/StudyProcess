首先需要了解java的命令：
javac :将java文件编译为.class文件，里面是一些二进制文件。
javap -c：将.class文件变为反汇编，例如
javap -c hello.class > demo.txt，可以将class文件转化为txt文件
学习虚拟机实际上是学习虚拟机的规范
## JVM主要有三个子系统：
一，类加载子系统
二，运行时数据区（内存结构）
三，执行引擎
### 一，类加载子系统
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407165915573.png)
#### 1，生命周期
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407203819390.png)
**加载**：把class二进制文件反编译到JVM内存模型中，并根据Class文件描述创建java.lang.Class对象的过程，类加载过程主要包含将Class文件读取到运行时区域的方法区，在堆中创建java.lang.Class对象，并且封装类在方法区的数据结构的过程。
**验证**：验证字节码的文件的正确性，文件格式的验证，字节码验证，符号引用的验证。
**准备**：在方法区中为类变量分配内存空间并设置类中变量的初始值。初始值根据不同的类型有不同的默认值，注意区分final类型跟非final类型的初始化过程是不同的。

```
public static long value  = 1000；
```
在以上代码中，静态变量value在准备阶段是0，将value设置为1000的动作是在对象的初始化的时候完成的，因为JVM在编译阶段会将静态变量的初始化操作定义在构造器中。但是如果是final类型的

```
public static final long value  = 1000;
```
在JVM的编译阶段就会为final类型的变量value赋值为1000；

**解析**：把类里面引用的其他类也加载进来，把符号引用转变为直接引用，也叫静态链接。
**初始化**：给静态变量一些真正的值，执行静态代码块。
**使用**：执行引擎
**卸载**：异常终止，操作系统错误，程序结束。
#### 2，类加载器的种类：
分为三种，分别是：
启动类加载器：负责加载java_HOME/lib目录中的类库，启动程序。
扩展类加载器：负责加载java_HOME/lib/ext目录中的类库，外部库。
应用程序类加载器：负责加载用户自己写的类库。
#### 3，双亲委派模型类加载机制

JVM通过双亲委派机制对类进行加载。双亲委派机制是指一个类在收到类加载的请求后不会尝试自己加载这个类，而是把该类加载请求向上委派给其父类去完成，其父类在接收到该类加载请求后又会将其委派给自己的父类，以此类推，这样所有的类加载请求都被向上委派到启动类加载器中。若父类加载器在接收到类加载请求后发现自己也无法加载该类（通常原因是该类的Class文件在父类的类加载器中不存在），则父类将会将该信息反馈给子类并向下委派子类加载器加载该类，直到该类被加载成功，若是找不到该类，则JVM会抛出ClassNotFoud异常。
具体流程如下所示：

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200722202748199.png)

## 二：内存结构
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407170332707.png)
#### 1，整体结构
其中堆和方法区是共享的，虚拟机栈，本地方法栈，程序计数器是私有的，线程的私有区域的生命周期是与线程相同的，随线程的启动而创建，随着线程的结束而销毁。对于私有的数据我们一般是管不到的，我们主要是对堆和方法区进行调优，对于共享区域是随虚拟机的启动而创建，随着虚拟机的关闭而销毁。
补充：
JVM还有一个直接内存，叫做堆外内存，他并不是JVM运行时数据区的一部分，但是在并发编程的时候频繁使用。
#### 2，程序计数器
程序计数器是用于存储当前运行的线程所执行的字节码的行号指示器。每个运行中的线程都有一个独立的程序计数器，在方法正在执行时，该方法的程序计数器记录的是实时虚拟机字节码指令的地址；如果该方法执行的是Native方法，则程序计数器的值为空。
#### 3，虚拟机栈----描述Java方法的执行
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407171127459.png)
局部变量：存放我们定义的局部变量
操作数栈：我们操作的代码
动态链接：把符号引用转换为直接引用。引用的类型一般存在堆中，
方法名以字符的形式保存在常量池中（Constant pool）。
方法出口：方法执行完成之后，指向调用者（调用方法)。

#### 4，本地方法栈
 java的底层是C语言，C++，一些底层代码是native方法，一些代码不需要自己去实现，只需要调用底层代码就可以了，在本地方法库中以.dll形式存在。通过调用本地方法栈中的库区执行底层代码就可以实现一些功能
 #### 5，方法区
 所定义的所有的方法信息都保存在该区域，存放的是静态变量，常量，类的信息，运行的常量池。
 #### 6， 堆（Heap）---运行时数据区
存放的是所有的对象，虚拟机在启动时为每一个对象分配空间，当对象内存空间不足时会跑出outOfMemoryError异常。
 注意：栈是运行的单位，堆是存取的单位。
 ![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2020040718114290.png)
 新生代：占了1/3的内存区域，类出生，成长，消亡的区域，一个类在这里产生，应用，最后被垃圾收集器收集（ygc）
 伊甸Eden区：8/10，
 From区：1/10
 To区：1/10
 From与To都属于幸存者（Survioror)区,这两个区域可以相互转化，等转化了15次就会转移到老年代。
 老年代：占了2/3的内存区域，存放的是比较大，存活时间比较长的实例对象。进行垃圾收集为fgc

## 三、执行引擎
分为即时编译器和垃圾回收器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407170228465.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

### 怎样判断对象可以被回收？
两种方法，一是引用计数器法，而是可达性分析法。
引用计数器法，是给对象添加计数器，如果引用了一个对象一次计数器加一，如果引用没有了就减一，直到为0的时候，这个时候分配不到内存了，就会发生一次gc，这时候把计数器为0的对象统一回收，方法比较简单有效，但是有个致命缺点，如果两个对象互相持有对方的引用，这个时候永远不会回收。
可达性分析算法，从gc root向下搜索，搜索走过的路径叫做引用面，如果	一个对象没有任何一个引用面的时候，就是一个不可达对象，就可以被回收。
gc root：类加载器，Thread，虚拟机的局部变量，static成员，本地方法栈的变量等。

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407211432162.png)
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407211602526.png)

### 如何判断一个常量是废弃常量？
没有任何类型指向这个常量，就被判断为废弃常量，就会回收。

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407213027381.png)
比较重要的是复制算法，标记清除算法，标记整理算法。
标记清除算法：首先标记需要回收的对象，在标记完成后统一回收标记的对象，它有两个不足，一是效率问题，标记和清除的效率都不高，标记的时候每一个都需要标记，清除的时候每一个对象都需要判断；二是空间问题，标记清除后会产生大量的不连续碎片。

基于效率问题，产生了一个复制算法。
**复制算法**：把内存大小分为相同的两块，每次只使用其中的一块，把这一块的还存活的对象复制的另一块上，然后把使用空间的一块一次性清除。只是这种方法将可使用内存缩小了一半未免太高了点。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407214046215.png)

  现在的商业虚拟机都采用复制算法来回收新生代，IBM公司的专门表明，新生代中的对象98%是"朝生夕死"的，所以并不需要按照1:1的比例来划分内存空间，而是将内存分为一块较大的Enden空间和两块较小的Survivor空间，当回收的时候将Eden和Survivor空间，每次使用Eden和其中一块Survivor，当回收时，将Eden和Survivor中还存活的对象一次性复制到另一块Survivor空间上，最后清理掉Eden和刚才用过的Survivor空间。HotSpot虚拟机默认的Eden和Survior的大小比例是8:1，也就是每次新生代可用的内存空间为这个歌新生代容量的90%，只有10%的内存会被“浪费"。当然，我们没法保证每次回收都只有差不多10%的对象被回收，所以需要依赖其他内存（指老年代）进行分配担保（Handle Promotion）。
  **标记整理法**：
   复制整理算法在存活对象存活率比较高时，效率将会降低。更关键的是如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用内存中所有对象都100%存活的极端情况，所以在老年代不再使用复制算法。
  根据老年代的特点，提出了”标记-整理算法“，标记过程任然与标记整理算法一样，但后续步骤不是直接对可回收对象进行整理，而是让所有存活对象都向一端移动，然后直接清理掉两端边界以外的内存。
  ![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407222151530.png)
**分代算法**：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407222541544.png)

### 垃圾收集器
如果说垃圾收集算法是方法论，那么垃圾收集器就是垃圾回收的具体实现。
目前比较新的垃圾收集器为gcc
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407223302257.png)
每一种项目所使用的垃圾收集器都是不一样的，现在目前一般使用的是G1。
Serial(串行)收集器，针对于过去CPU不太好，是一个单线程的收集器，把所有的任务都停掉，专门进行垃圾收集。因此停顿时间比较长，数据吞吐量比较低。虽然比较古老，但并不是不用了，对于桌面的用户来说，还是可以接受的。
PerNew收集器，实际上是对于Serial收集器的多线程版本，它的缺点跟Serial也是一样的。
Paraller Scavenge收集器，目前使用最多的收集器，类似于PerNew收集器的，它主要关注的是数据的吞吐量，它使用的算法是不一样的，新生代使用复制算法，老年代使用标记-整理算法，但是还是没有解决停顿的缺点。
CMS（Concurrent Mark Sweep）收集器，并发垃圾收集器，基于标记清除算法的收集器。它的停顿时间比较短，注重用户体验。
![!\[在这里插入图片描述\blog.csdnimg.cn/20200407224753544.png)](https://img-blog.csdnimg.cn/20200407225029859.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407225314542.png)
对CPU资源比较敏感：对CPU的资源要求比较高。
浮动垃圾：它在运行过程产生的垃圾。
G1收集器：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2020040722560752.png)
怎样选择收集器？
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2020040722582487.png)
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407225855675.png)
GC调优步骤
调优是在一开始就要调优的，
gc日志分析工具：GCeasy
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407230425377.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200407230047958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
补充
类加载机制：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200407151415910.png)