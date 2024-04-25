# ThreadLocal 
ThreadLocal 是 Java 中一个非常有用的类，它允许你创建线程局部变量。线程局部变量是指每个线程都有自己独立的变量副本，互不干扰。  
ThreadLocal 主要用于解决多线程环境下共享数据的线程安全性问题。
## 基本用法
创建 ThreadLocal 变量  
```
ThreadLocal<Integer> threadLocalVariable = new ThreadLocal<>();
```
设置和获取线程局部变量：  
```
threadLocalVariable.set(42); // 设置线程局部变量的值
int value = threadLocalVariable.get(); // 获取线程局部变量的值
```
初始值：  
你可以通过覆盖 ThreadLocal 的 initialValue 方法来指定线程局部变量的初始值。例如：    
```
ThreadLocal<Integer> threadLocalVariable = ThreadLocal.withInitial(() -> 0);
```
注意事项：  
使用 ThreadLocal 时要小心内存泄漏，确保在不需要使用线程局部变量时及时清理。   
在使用线程池时，注意线程复用可能导致线程局部变量的状态被共享。  
ThreadLocal 是一个有助于在多线程应用程序中维护线程局部状态的重要工具，但它需要谨慎使用，以避免潜在的问题。确保理解其工作原理，并在需要的情况下适当使用它，可以提高多线程程序的性能和可维护性。  
## ThreadLocal理解
ThreadLocal是一个将在多线程中为每一个线程创建单独的变量副本的类; 当使用ThreadLocal来维护变量时, ThreadLocal会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况。  
最多的是session管理和数据库链接管理，这里以数据访问为例帮助你理解ThreadLocal：    
```
class ConnectionManager {
    private static Connection connect = null;

    public static Connection openConnection() {
        if (connect == null) {
            connect = DriverManager.getConnection();
        }
        return connect;
    }

    public static void closeConnection() {
        if (connect != null)
            connect.close();
    }
}
```
据库管理类在单线程使用是没有任何问题的，在多线程中使用会存在线程安全问题：
第一，这里面的2个方法都没有进行同步，很可能在openConnection方法中会多次创建connect；  
第二，多次进行closeConnection可能会导致空指针异常问题；  
第三，由于connect是共享变量，那么必然在调用connect的地方需要使用到同步来保障线程安全，因为很可能一个线程在使用connect进行数据库操作，而另外一个线程调用closeConnection关闭链接。    

为了解决上述线程安全的问题，第一考虑：**互斥同步**  
你可能会说，将这段代码的两个方法进行同步处理，并且在调用connect的地方需要进行同步处理，比如用Synchronized或者ReentrantLock互斥锁。  
这里再抛出一个问题：这地方到底需不需要将connect变量进行共享?  
事实上，是不需要的。假如每个线程中都有一个connect变量，各个线程之间对connect变量的访问实际上是没有依赖关系的，即一个线程不需要关心其他线程是否对这个connect进行了修改的。即改后的代码可以这样：  
```
class ConnectionManager {
    private Connection connect = null;

    public Connection openConnection() {
        if (connect == null) {
            connect = DriverManager.getConnection();
        }
        return connect;
    }

    public void closeConnection() {
        if (connect != null)
            connect.close();
    }
}

class Dao {
    public void insert() {
        ConnectionManager connectionManager = new ConnectionManager();
        Connection connection = connectionManager.openConnection();

        // 使用connection进行操作

        connectionManager.closeConnection();
    }
}
```
这样处理确实也没有任何问题，由于每次都是在方法内部创建的连接，那么线程之间自然不存在线程安全问题。但是这样会有一个致命的影响：导致服务器压力非常大，并且严重影响程序执行性能。由于在方法中需要频繁地开启和关闭数据库连接，这样不仅严重影响程序执行效率，还可能导致服务器压力巨大。  
这时候ThreadLocal登场了那么这种情况下使用ThreadLocal是再适合不过的了，因为ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。下面就是网上出现最多的例子：  
```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionManager {

    private static final ThreadLocal<Connection> dbConnectionLocal = new ThreadLocal<Connection>() {
        @Override
        protected Connection initialValue() {
            try {
                return DriverManager.getConnection("", "", "");
            } catch (SQLException e) {
                e.printStackTrace();
            }
            return null;
        }
    };

    public Connection getConnection() {
        return dbConnectionLocal.get();
    }
}
```
## ThreadLocal原理

### 如何实现线程隔离
主要是用到了Thread对象中的一个ThreadLocalMap类型的变量threadLocals, 负责存储当前线程的关于Connection的对象, dbConnectionLocal(以上述例子中为例) 这个变量为Key, 以新建的Connection对象为Value; 这样的话, 线程第一次读取的时候如果不存在就会调用ThreadLocal的initialValue方法创建一个Connection对象并且返回;    
具体关于为线程分配变量副本的代码如下:  
```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap threadLocals = getMap(t);
    if (threadLocals != null) {
        ThreadLocalMap.Entry e = threadLocals.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

##  ThreadLocal 内存泄漏
### 原因  
ThreadLocal 内存泄漏是指在多线程环境中，由于 ThreadLocal 实例被长时间持有，而没有适时地进行清理或者移除，导致相关线程持有的 ThreadLocalMap 中的 Entry 无法被及时回收，从而导致内存占用持续增加，最终可能导致内存溢出。  

ThreadLocalMap 是 ThreadLocal 类的一个私有静态内部类，它用于存储每个线程的 ThreadLocal 变量和其对应的值。每个线程在使用 ThreadLocal 时都会在自己的 ThreadLocalMap 中维护一份该 ThreadLocal 实例和对应值的映射。当线程结束时，ThreadLocalMap 中的 Entry 应当被及时清理，以释放相关资源，但如果没有适时地清理 ThreadLocalMap 中的 Entry，就会发生内存泄漏。  

ThreadLocal 内存泄漏的常见原因包括：  

1、线程池未及时清理： 在使用线程池时，如果线程池中的线程长时间持有 ThreadLocal 实例而不被释放，那么 ThreadLocalMap 中的 Entry 就会一直存在，从而导致内存泄漏。  

2、长时间持有 ThreadLocal 实例： 如果一个 ThreadLocal 实例被长时间持有，并且在该实例的生命周期内频繁创建线程，那么每个线程都会在 ThreadLocalMap 中添加对应的 Entry，如果不及时移除这些 Entry，就会导致内存泄漏。  

3、未手动移除 ThreadLocal 实例： 在使用 ThreadLocal 时，如果不手动调用 remove() 方法来移除对应的 ThreadLocal 实例，就会导致 ThreadLocalMap 中的 Entry 无法及时清理。  

4、为了避免 ThreadLocal 内存泄漏，应该在使用完 ThreadLocal 实例后及时调用 remove() 方法来手动移除对应的 ThreadLocal 实例，或者使用 ThreadLocal 的 withInitial() 方法来创建 ThreadLocal 实例，并确保每次使用完 ThreadLocal 实例后将其设置为 null，以便帮助垃圾回收器识别并及时回收不再需要的对象。此外，对于线程池中的线程，应该确保在线程执行完毕后清理 ThreadLocal 实例。  

### 避免 ThreadLocal 内存泄漏
手动移除 ThreadLocal 实例： 在使用完 ThreadLocal 实例后，及时调用 remove() 方法来手动移除对应的 ThreadLocal 实例。这样可以确保  
1、ThreadLocalMap 中的 Entry 在不再需要时能够及时清理，避免内存泄漏。    
```
threadLocal.remove();
```
2、使用 initialValue 方法或者 withInitial 方法初始化 ThreadLocal 实例： 可以使用 initialValue 方法或者 withInitial 方法来初始化 ThreadLocal 实例，这样可以确保每个线程都有一个独立的初始化值，而不会共享相同的初始值。
这样可以减少对 ThreadLocalMap 的使用，从而降低内存泄漏的风险。  
```
ThreadLocal<String> threadLocal = ThreadLocal.withInitial(() -> "initialValue");
```
3、使用弱引用（WeakReference）来包装 ThreadLocal 实例： 可以使用弱引用来包装 ThreadLocal 实例，这样一旦 ThreadLocal 实例没有强引用指向它，就可以被垃圾回收器回收。但需要注意的是，这样做可能会影响性能，因为弱引用需要更频繁地进行垃圾回收。  
```
ThreadLocal<String> threadLocal = new ThreadLocal<>();
WeakReference<ThreadLocal<String>> weakReference = new WeakReference<>(threadLocal);
```
4、使用 InheritableThreadLocal 代替 ThreadLocal（在需要线程间继承值的场景下）： InheritableThreadLocal 是 ThreadLocal 的一个子类，它可以让子线程从父线程中继承值。在某些场景下，使用 InheritableThreadLocal 可以避免一些潜在的内存泄漏问题，但需要注意，InheritableThreadLocal 会增加一定的开销。  
```
InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();
```
总之，通过以上方法，可以有效地避免 ThreadLocal 内存泄漏问题，确保在多线程环境下正常使用 ThreadLocal，并有效管理 ThreadLocalMap 中的 Entry。  
## 底层ThreadLocalmap  

ThreadLocalMap 是 ThreadLocal 类的一个内部类，它用于存储每个线程的 ThreadLocal 变量和其对应的值。每个线程在使用 ThreadLocal 时都会在自己的 ThreadLocalMap 中维护一份 ThreadLocal 实例和对应值的映射关系。  

ThreadLocalMap 是基于开放地址法的哈希表实现的，它包含一个固定大小的 Entry[] 数组，用于存储 ThreadLocal 实例和对应值的映射关系。哈希表的索引是通过 ThreadLocal 实例的哈希值计算得到的，每个 Entry 包含一个 ThreadLocal 实例和对应的值。  

由于每个线程都拥有自己的 ThreadLocalMap 实例，因此 ThreadLocalMap 的访问是线程安全的，不需要额外的同步措施。这样可以确保每个线程都只能访问自己的 ThreadLocal 变量和对应的值，避免了多线程并发访问时的竞态条件。  

需要注意的是，ThreadLocalMap 中的 Entry 对象是使用弱引用来引用 ThreadLocal 实例的，这样可以避免 ThreadLocal 实例被 ThreadLocalMap 长期持有而导致的内存泄漏。当 ThreadLocal 实例没有其他强引用指向它时，它会被垃圾回收器回收，相应的 Entry 也会被自动清理。  

总之，ThreadLocalMap 是 ThreadLocal 实现线程隔离的核心机制之一，它通过为每个线程维护一份 ThreadLocal 变量和对应值的映射关系来实现线程隔离，从而确保了多线程环境下的安全访问。  













https://pdai.tech/md/java/thread/java-thread-x-threadlocal.html








https://pdai.tech/md/java/thread/java-thread-x-threadlocal.html
