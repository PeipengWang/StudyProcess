# final的安全发布

两个关键字“发布”“安全”    
所谓发布通俗一点的理解就是创建一个对象，使这个对象能被当前范围之外的代码所使用    
比如Object o = new Object();    
然后接下来使用对象o    
但是对于普通变量的创建，之前分析过，大致分为三个步骤：    
1、分配内存空间    
2、将o指向分配的内存空间    
3、调用构造函数来初始化对象    
这三个步骤不是原子的，如果执行到第二步，还没有进行初始化，此时对象已经不是null了，如果被其他代码访问，这将收获一个错误的结果。  
或者说对象尚未完全创建就被使用了，其他线程看到的结果可能是不一致的，这就是不安全的发布
根本原因就是JVM创建对象的过程涉及到分配空间、指针设置、数据初始化等步骤，并不是同步的，涉及到主存与缓存、处理器与寄存器等，可见性没办法得到保障  
所以说，什么是安全发布，简单理解就是对象的创建能够保障在被别人使用前，已经完成了数据的构造设置，或者说一个对象在使用时，已经完成了初始化。  
不幸的是，Java对此并没有进行保障，你需要自己进行保障，比如synchronized关键字，原子性、排他性就可以做到这一点  
## 不安全的发布实例

## 怎么保障安全发布？有几种方法：
一种是刚才提到的锁机制，通过加锁可以保障中间状态不会被读取  
另外还有：   
1、借助于volatile或者AtomicReference声明对象  
2、借助于final关键字  
3、在静态初始化块中，进行初始化（JVM会保障）  
4、将对象引用保存到一个由锁保护的域中  
5、借助AtomicReference  

很显然，对于锁机制，那些线程安全的容器比如ConcurrentMap，也是满足这条的，所以也是安全发布  
对于final，当你创建一个对象时，使用final关键字能够使得另一个线程不会访问到处于“部分创建”的对象  
因为：当构造函数退出时，final字段的值保证对访问构造对象的其他线程可见  
如果某个成员是final的，JVM规范做出如下明确的保证：  
一旦对象引用对其他线程可见，则其final成员也必须正确的赋值  
所以说借助于final，就如同你对对象的创建访问加锁了一般，天然的就保障了对象的安全发布。  
对于普通的变量，对象的内存空间分配、指针设置、数据初始化，和将这个变量的引用赋值给另一个引用，之间是可能发生重排序的，所以也就导致了其他线程可能读取到不一致的中间状态  
但是对于final修饰的变量，JVM会保障顺序  
不会在对final变量的写操作完成之前，与将变量引用赋值给其他变量之间进行重排序，也就是final变量的设置完成始终会在被读取之前  
final除了不可变的定义之外，还与线程安全发布息息相关  
借助于final，可以达到对象安全发布的保障，只需要借助于final，不在需要任何额外的付出，他能够保障在多线程环境下，总是能够读取到正确的初始化的值  
所以，如果你不希望变量后续被修改，你应该总是使用final关键字  
而且，很显然在某些场景下，final也可以解决一定的安全问题  
## 实例
使用synchronized锁的时候，作为锁的对象最好要加上final修饰符，因为可能线程会改变锁变量持有的具体的对象。  
demo如下：  

public class Test02 {
    static Object lock = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            lock = new Object();
            synchronized (lock) {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(5);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("A");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            lock = new Object();
            synchronized (lock) {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(5);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("B");
                }
            }
        });

        t1.start();
        t2.start();
    }
}

但是要是把锁改成final的。代码如下：  
```
public class Test02 {
    static final Object lock = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
//            lock = new Object(); // 编译出错，final不能修改
            synchronized (lock) {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(5);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("A");
                }
            }
        });

        Thread t2 = new Thread(() -> {
//            lock = new Object();
            synchronized (lock) {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(5);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("B");
                }
            }
        });

        t1.start();
        t2.start();
    }
}

```
实例参考：https://juejin.cn/post/7104070219806539806
原理参考：https://www.cnblogs.com/noteless/p/10416678.html
