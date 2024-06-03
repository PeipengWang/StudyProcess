## InheritableThreadLocal实现原理

为什么线程池的线程无法获取业务线程存放到InheritableThreadLocal的数据？这就得从InheritableThreadLocal的原理说起了。    

 InheritableThreadLocal 继承自ThreadLocal，重写了其中crateMap方法和getMap方法。重写这两个方法的目的是使得所有线程通过InheritableThreadLocal设置的上下文信息，都保存在其对应的inheritableThreadLocals属性中。这一点和ThreadLocal不同，ThreadLocal是保存在Thread的threadLocals属性中。  
 下面是Thread类汇中threadLocals属性和inheritableThreadLocals两个属性,当调用ThreadLocal类型的上下文对象设置参数时，设置的就是其threadLocals属性对应的Map的kv值，当调用InheritableThreadLocal类型的上下文对象设置参数时，就是设置其inheritableThreadLocals属性的kv值：

```java
 /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

下面是InheritableThreadLocal重写的crateMap方法和getMap方法，正是通过这两个方法，改变了ThreadLocal中要设置和获取Thread中哪个属性的方法。

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    /**
     * Computes the child's initial value for this inheritable thread-local
     * variable as a function of the parent's value at the time the child
     * thread is created.  This method is called from within the parent
     * thread before the child is started.
     * <p>
     * This method merely returns its input argument, and should be overridden
     * if a different behavior is desired.
     *
     * @param parentValue the parent thread's value
     * @return the child thread's initial value
     */
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * Get the map associated with a ThreadLocal.
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}

```

那么子线程是如何能获取到父线程保存到InheritableThreadLocal类型上下文中数据的呢？  
 原来是在创建Thread对象时，会判断父线程中inheritableThreadLocals是否不为空，如果不为空，则会将父线程中inheritableThreadLocals中的数据复制到自己的inheritableThreadLocals中。这样就实现了父线程和子线程的上下文传递。     

```java
public Thread(ThreadGroup group, Runnable target, String name,
              long stackSize) {
    init(group, target, name, stackSize);
}

private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize) {
    init(g, target, name, stackSize, null, true);
}

private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }

    this.name = name;
    ............
    ............
    // 此处会初始化Thread对应的inheritThreadLocals
    // 如果父线程的inheritThreadLocals不为空，则会复制父线程的inheritThreadLocals
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        // 根据父线程的inheritableThreadLocals对应的parentMap创建子线程的inheritableThreadLocals
        this.inheritableThreadLocals = ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize;

    /* Set thread ID */
    tid = nextThreadID();
}
// 根据父线程的inheritableThreadLocals对应的parentMap创建子线程的inheritableThreadLocals 
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
    return new ThreadLocalMap(parentMap);
}


private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];
    // 将父Thread中的inheritThreadLocals（ThreadLocalMap）对应的的所有值都复制给子类的inheritThreadLocals
    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                // 注意此处的childValue()方法对应InheritableThreadLocal中重写的childValue方法
                // 在前面InheritableThreadLocal中，是直接返回了传入的值
                // 也就是直接将这个值的引用（引用类型时是对象的引用，普通类型或者String时，传的是值本身）传给了子线程的Entry
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}

```

通过上面对InheritableThreadLocal的简单分析，我们可以知道，在创建Thread时，才会将父线程中的inheritableThreadLocals复制给新创建Thread的inheritableThreadLocals。  
 但是在线程池中，业务线程只是将任务对象（实现了Runnable或者Callable的对象）加入到任务队列中，并不是去创建线程池中的线程，因此线程池中线程也就获取不到业务线程中的上下文信息。  
 那么在线程池的场景下就没有方法解决了吗，只能去修改业务代码，在提交任务对象时，手工传入buzSource吗？观察了一下，项目中，前人写的代码，传个traceId啥的，都是这么修改业务代码传入traceId之类参数，这样很不优雅，如果以后有其他类似的字段，还是需要大量的进行修改，有没有什么办法可以解决这个问题？    

## 解决遇到线程池InheritableThreadLocal就废了的问题

### 方法一：自定义实现

其实仔细想想，子线程之所以能获得父线程放到InheritableThreadLocal的数据，是因为在创建子线程时，复制了父线程的inheritableThreadLocals属性，触发复制的时机是创建子线程的时候。

在线程池场景下，是提交任务，既然要提交任务，那么就要创建任务，那么能否在创建任务的时候，做做文章呢？

下面是我的实现：

1、定义一个InheritableTask抽象类，这个类实现了Runaable接口，并定义了一个runTask抽象方法，当开发者需要面对线程池获取InheritableThreadLocal值的场景时，提交的任务对象，只需要继承InheritableTask类，实现runTask方法即可。

2、在创建任务类时，也就是在InheritableTask构造函数中，通过反射获，获取到提交任务的业务线程的inheritableThreadLocals属性，然后复制一份，暂存到当前task的inheritableThreadLocalsObj属性中。

3、线程池线程在执行该任务时，其实就是去调用其run()方法，在执行run方法时，先将暂存的inheritableThreadLocalsObj属性，赋值给当前执行任务的线程，这样这个线程就可以得到提交任务的那个业务线程的inheritableThreadLocals属性值了。然后再去执行runTask(),就是真正的业务逻辑。最后，finally清理掉执行当前业务的线程的inheritableThreadLocals属性。

```java
/**
 * @author 王二北
 * @description
 * @date 2019/8/21
 */
public abstract class InheritableTask implements Runnable {
    private Object inheritableThreadLocalsObj;
    public InheritableTask(){
       try{
           // 获取业务线程的中的inheritableThreadLocals属性值
           Thread currentThread = Thread.currentThread();
           Field inheritableThreadLocalsField = Thread.class.getDeclaredField("inheritableThreadLocals");
           inheritableThreadLocalsField.setAccessible(true);
           // 得到当前线程中的inheritableThreadLocals属性值
           Object threadLocalMapObj = inheritableThreadLocalsField.get(currentThread);
           if(threadLocalMapObj != null){
               // 调用ThreadLocal中的createInheritedMap方法，重新复制一个新的inheritableThreadLocals值
              Class threadLocalMapClazz =   inheritableThreadLocalsField.getType();
               Method method =  ThreadLocal.class.getDeclaredMethod("createInheritedMap",threadLocalMapClazz);
               method.setAccessible(true);
              // 创建一个新的ThreadLocalMap类型的inheritableThreadLocals
              Object newThreadLocalMap = method.invoke(ThreadLocal.class,threadLocalMapObj);
              // 将这个值暂存下来
              inheritableThreadLocalsObj = newThreadLocalMap;
           }
       }catch (Exception e){
           throw new IllegalStateException(e);
       }
    }

    /**
     * 搞个代理方法，这个方法中处理业务逻辑
     */
    public abstract void runTask();

    @Override
    public void run() {
        // 此处得到的是当前处理该业务的线程，也就是线程池中的线程
        Thread currentThread = Thread.currentThread();
        Field field = null;
        try {
            field  = Thread.class.getDeclaredField("inheritableThreadLocals");
            field.setAccessible(true);
            // 将暂存的值，赋值给currentThread
            if (inheritableThreadLocalsObj != null && field != null) {
                field.set(currentThread, inheritableThreadLocalsObj);
                inheritableThreadLocalsObj = null;
            }
            // 执行任务
            runTask();
        }catch (Exception e){
            throw new IllegalStateException(e);
        }finally {
            // 最后将线程中的InheritableThreadLocals设置为null
           try{
               field.set(currentThread,null);
           }catch (Exception e){
               throw new IllegalStateException(e);
           }
        }
    }
}
```

下面做个例子测试一下：

```java
public class TestInheritableThreadLocal {
    private static InheritableThreadLocal<String> local = new InheritableThreadLocal();
    private static ExecutorService es = Executors.newFixedThreadPool(5);
    public static void main(String[] args)throws Exception{
        for(int i =0;i<2;i++){
            final int ab = i;
            new Thread(){
                public void run(){
                    local.set("task____"+ab);
                    for(int i = 0;i<3;i++){
                        final  int a = i;
                        es.execute(new InheritableTask() {
                            @Override
                            public void runTask() {
                              System.out.println(Thread.currentThread().getName()+"_"+ ab+"get_"+ a +":" + local.get());
                            }
                        });
                    }
                }
            }.start();
        }
    }
)
运行结果，每个线程设置的值，都能被正确的获取到：
pool-1-thread-3_0get_1:task____0
pool-1-thread-4_1get_1:task____1
pool-1-thread-5_0get_2:task____0
pool-1-thread-1_1get_0:task____1
pool-1-thread-2_0get_0:task____0
pool-1-thread-3_1get_2:task____1
```



这样，就解决了在线程池场景下InheritableThreadLocal无效的问题。  
 然而，就这么完了吗？不，别忘了，反射是比较耗性能的。  
 一般优化反射性能的方式有两种，一种是使用缓存，一种是使用性能较高的反射工具，比如RefelectASM之类的。  
 我再使用的时候回发现RefelectAsm并不是特别好用，因为其不能反射获取private的字段，并且在获取inheritableThreadLocals字段时总是不成功，这里只展示一下使用缓存的实现：  

```java
/**
 * @author 王二北
 * @description
 * @date 2019/8/21
 */
public abstract class InheritableTaskWithCache implements Runnable {
    private Object obj;
    private static volatile Field inheritableThreadLocalsField;
    private static volatile Class threadLocalMapClazz;
    private static volatile Method createInheritedMapMethod;
    private static final Object accessLock = new Object();


    public InheritableTaskWithCache(){
       try{
           Thread currentThread = Thread.currentThread();
           Field field = getInheritableThreadLocalsField();
           // 得到当前线程中的inheritableThreadLocals熟悉值ThreadLocalMap, key是各种inheritableThreadLocal，value是值
           Object threadLocalMapObj = field.get(currentThread);
           if(threadLocalMapObj != null){
              Class threadLocalMapClazz = getThreadLocalMapClazz();
              Method method =  getCreateInheritedMapMethod(threadLocalMapClazz);
              // 创建一个新的ThreadLocalMap
              Object newThreadLocalMap = method.invoke(ThreadLocal.class,threadLocalMapObj);
              obj = newThreadLocalMap;
           }
       }catch (Exception e){
           throw new IllegalStateException(e);
       }
    }

    private Class getThreadLocalMapClazz(){
        if(inheritableThreadLocalsField == null){
            return null;
        }else {
            if(threadLocalMapClazz == null){
                synchronized (accessLock){
                    if(threadLocalMapClazz == null){
                        Class clazz = inheritableThreadLocalsField.getType();
                        threadLocalMapClazz = clazz;
                    }
                }
            }
        }
        return threadLocalMapClazz;
    }

    private Field getInheritableThreadLocalsField(){
        if(inheritableThreadLocalsField == null){
            synchronized (accessLock){
                if(inheritableThreadLocalsField == null){
                    try {
                        Field field = Thread.class.getDeclaredField("inheritableThreadLocals");
                        field.setAccessible(true);
                        inheritableThreadLocalsField = field;
                    }catch (Exception e){
                        throw new IllegalStateException(e);
                    }
                }
            }
        }
        return inheritableThreadLocalsField;
    }

    private Method getCreateInheritedMapMethod(Class threadLocalMapClazz){
        if(threadLocalMapClazz != null && createInheritedMapMethod == null){
            synchronized (accessLock){
                if(createInheritedMapMethod == null){
                    try {
                        Method method =  ThreadLocal.class.getDeclaredMethod("createInheritedMap",threadLocalMapClazz);
                        method.setAccessible(true);
                        createInheritedMapMethod = method;
                    }catch (Exception e){
                        throw new IllegalStateException(e);
                    }
                }
            }
        }
        return createInheritedMapMethod;
    }

    public abstract void runTask();

    @Override
    public void run() {
        boolean isSet = false;
        Thread currentThread = Thread.currentThread();
        Field field = getInheritableThreadLocalsField();
        try {
            if (obj != null && field != null) {
                field.set(currentThread, obj);
                obj = null;
                isSet = true;
            }
            // 执行任务
            runTask();
        }catch (Exception e){
            throw new IllegalStateException(e);
        }finally {
            // 最后将线程中的InheritableThreadLocals设置为null
           try{
               field.set(currentThread,null);
           }catch (Exception e){
               throw new IllegalStateException(e);
           }
        }
    }
}
```

下面对比一下使用缓存和不使用缓存的性能：

我使用的笔记本电脑，是I7,8核16G， 测试时，由于已经开了几个idea和一堆Chrome网页，cpu使用率已经达到60%左右。

首先是不使用缓存直接反射的Task实现，共执行了6次，每次都创建了3000w个InheritableTask对象，每次执行耗时如下：

```
reflect cost:2905ms
reflect cost:2165ms
reflect cost:2424ms
reflect cost:2756ms
reflect cost:2242ms
reflect cost:2487ms
```

然后是使用缓存反射字段的task实现，共执行了6次，每次都创建了3000w个InheritableTask对象，每次执行耗时如下：

```
cache cost:82ms
cache cost:70ms
cache cost:58ms
cache cost:94ms
cache cost:71ms
cache cost:60ms
```

可以发现使用cache的情况下，性能提高了30~40倍。总的来说，在使用缓存的情况下，性能还是不错的。

综上，通过实现一个抽象的InheritableTask解决了线程池场景下InheritableThreadLocal“失效”的问题。

总结：
 1、InheritableThreadLocal在线程池下是无效的，原因是只有在创建Thread时才会去复制父线程存放在InheritableThreadLocal中的值，而线程池场景下，主业务线程仅仅是将提交任务到任务队列中。

 2、如果需要解决这个问题，可以自定义一个RunTask类，使用反射加代理的方式来实现业务主线程存放在InheritableThreadLocal中值的间接复制。

### 方法二：TransmittableThreadLocal

TTL使用：
**1、添加pom:**

```xml
		<dependency>
             <groupId>com.alibaba</groupId>
             <artifactId>transmittable-thread-local</artifactId>
             <version>2.12.0</version>
         </dependency>
```

示例：1和5处代码变化，添加6。把具体任务使用TtlRunnable.get(task)包装了下，然后再提交到线程池。

```java
public class Ttl {
 // 0.创建线程池
 private static final ThreadPoolExecutor bizPoolExecutor =
         new ThreadPoolExecutor(2, 2, 1, MINUTES,
         new LinkedBlockingQueue<>(1));
 
     public static void main(String[] args) throws InterruptedException {
 
         // 1 创建线程变量
         ThreadLocal<String> parent = new TransmittableThreadLocal<>();
 
         // 2 投递三个任务，让线程池中的线程全部创建。
         for (int i = 0; i < 3; ++i) {
             bizPoolExecutor.execute(() -> {
                 try {
                     Thread.sleep(3000);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
             });
         }
         // 3休眠4s
         Thread.sleep(4000);
         // 4.设置线程变量
         parent.set("value-set-in-parent");
 
         // 5. 提交任务到线程池
         Runnable task = () -> {
             try {
                 // 5.1访问线程变量
                 System.out.println("parent:" + parent.get());
             } catch (Exception e) {
                 e.printStackTrace();
             }
         };
 
         // 6、额外的处理，生成修饰了的对象ttlRunnable
         Runnable ttlRunnable = TtlRunnable.get(task);
         bizPoolExecutor.execute(ttlRunnable);
     }
 }
```

运行结果：

```
parent:value-set-in-parent
```

TransmittableThreadLocal 完美解决了线程变量继承问题

TTL解析：
在线程池场景下我们的需求：

InheritableThreadLocal 的是在 new Thread 时候把父线程的 inheritableThreadLocals 复制到了子线程，从而实现线程变量的继承特性。

而现在我们需要在提交任务到线程池前，把父线程中的线程变量保存到任务内，然后等线程池内线程执行任务前把保存的父线程的线程变量复制到线程池中的执行线程上，然后运行我们的任务，等任务运行完毕后，在清除掉执行线程上的线程变量。

可知第一我们需要包装提交到线程池内的任务，里面添加一个变量来保存父线程的线程变量。TransmittableThreadLocal 就是这样的思路。

如上代码6就是把我们的任务使用 TtlRunnable.get(task) 包装了，其内部就是拷贝父线程中的线程变量到包装的任务内保存起来。TtlRunnable.get(task)的get方法最后调到了：

```java
private TtlRunnable(Runnable runnable, boolean releaseTtlValueReferenceAfterRun) {  
     this.capturedRef = new AtomicReference<Object>(capture());  
     this.runnable = runnable;   
     this.releaseTtlValueReferenceAfterRun = releaseTtlValueReferenceAfterRun;   
 }
```

原理：

1、TTL 继承自 InheritableThreadLocal。
2、通过一个 Holder，保存了每个线程当前持有的所有 ThreadLocal 对象。
3、用 TtlRunnable 的 get 方法来包裹一个 Runnable 对象，包裹对象时，会采用类似 SNAPSHOT，快照的机制，通过 Holder，捕获父线程当前持有的所有ThreadLocal。随后，子线程启动，在Runnable对象执行run方法之前，从Holder中取出先前捕获到的父线程所持有的ThreadLocal对象，并设置到当前子线程当中，设置之前会保存子线程原有的ThreadLocal作为backUp，当子线程执行结束后，通过backUp恢复其原有的ThreadLocal。



https://www.jianshu.com/p/29f4034f4250
