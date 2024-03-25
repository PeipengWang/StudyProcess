# CMS垃圾回收
CMS GC的官方名称为“Mostly Concurrenct Mark and Sweep Garbage Collector”（最大-并发-标记-清除-垃圾收集器）。
作用范围： 老年代  
算法： 并发标记清除算法。  
启用参数：-XX:+UseConMarkSweepGC  
默认回收线程数：（处理器核心数量 + 3）/4Java9之后使用CMS垃圾收集器后，默认年轻代就为ParNew收集器，并且不可更改，同时JDK9之后被标记为不推荐使用，JDK14就被删除了。  
设计目标/优点：避免在老年代垃圾收集时出现长时间的卡顿，主要通过两种手段来达成此目标：  
	第一，不对老年代进行整理，而是使用空闲列表（free-list）来管理内存空间的回收  
	第二，在mark-and-sweep（标记-清除）阶段的大部分工作和应用线程一起并发执行。  

## CMS的几个阶段
CMS 处理过程有七个步骤：  
初始标记,会导致stw;  
并发标记，与用户线程同时运行；  
预清理，与用户线程同时运行；  
可被终止的预清理，与用户线程同时运行；  
重新标记 ，会导致stw；  
并发清除，与用户线程同时运行；  

或者说简化四个步骤  
初始标记：仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。  
并发标记：进行GC Roots Tracing的过程，在整个过程中耗时最长。  
重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。此阶段也需要“Stop The World”。  
并发清除。  

## 触发条件
1、如果没有设置-XX:+UseCMSInitiatingOccupancyOnly，虚拟机会根据收集的数据决定是否触发（建议线上环境带上这个参数，不然会加大问题排查的难度）。   
2、老年代使用率达到阈值 CMSInitiatingOccupancyFraction，默认92%。    
3、永久代的使用率达到阈值 CMSInitiatingPermOccupancyFraction，默认92%，前提是开启 CMSClassUnloadingEnabled。    
4、新生代的晋升担保失败。   
CMS的初始标记阶段，会扫描新生代对象，并把新生代的有效对象作为GC ROOT，用于第二阶段的扫描。   

因为老年代对象可能只被新生代对象引用，故需要扫描新生代  

## 问题
1、垃圾碎片的问题  
我们都知道CMS是使用的是标记-清除算法的，所以不可避免的就是会出现垃圾碎片的问题。  
2、一般CMS的GC耗时80%都在remark阶段，remark阶段停顿时间会很长  
3、concurrent mode failure  
发生在cms正在回收的时候。执行CMS GC的过程中，同时业务线程也在运行，当年轻带空间满了，执行ygc时，需要将存活的对象放入到老年代，而此时老年代空间不足，这时CMS还没有机会回收老年带产生的，或者在做Minor GC的时候，新生代救助空间放不下，需要放入老年代，而老年代也放不下而产生的。  
4、promotion failed  
进行Minor GC时，Survivor空间不足，对象只能放入老年代，而此时老年代也放不下造成的，多数是由于老年代有足够的空闲空间，但是由于碎片较多，新生代要转移到老年带的对象比较大,找不到一段连续区域存放这个对象导致的。  
解决方法：  
垃圾碎片问题：需要用到这个参数：-XX:CMSFullGCsBeforeCompaction=n， 意思是说在上一次CMS并发GC执行过后，到底还要再执行多少次full GC才会做压缩。默认是0，也就是在默认配置下每次CMS GC顶不住了而要转入full GC的时候都会做压缩。  

concurrent mode failure问题：-XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=60  
是指设定CMS在对内存占用率达到60%的时候开始GC。  
为什么设置这两个参数呢？由于在垃圾收集阶段用户线程还需要运行，那也就还需要预留有足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集。  

remark阶段停顿时间会很长的问题：解决这个问题巨简单，加入-XX:+CMSScavengeBeforeRemark。在执行remark操作之前先做一次Young GC，目的在于减少年轻代对老年代的无效引用，降低remark时的开销。  




