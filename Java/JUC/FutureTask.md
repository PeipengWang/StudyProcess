## FutureTask

是Java中的一个类，实现了RunnableFuture接口，该接口继承了Runnable和Future接口。FutureTask可以用来包装一个Callable或Runnable对象，使其具有异步计算的能力，同时允许取消任务、查询任务是否完成，以及获取任务的执行结果。
1、包装Callable或Runnable： FutureTask可以用来包装一个实现了Callable接口的任务或一个实现了Runnable接口的任务。
2、异步计算： 通过将FutureTask提交给Executor来异步执行任务。可以通过get()方法获取任务的执行结果，该方法会阻塞直到任务完成。
3、取消任务： 可以通过调用cancel()方法来取消任务的执行。一旦任务被取消，它就不能再次执行。
4、查询任务状态： 可以使用isDone()方法判断任务是否完成，以及使用isCancelled()方法判断任务是否被取消。
5、阻塞获取结果： get()方法可以用来获取任务的执行结果。如果任务尚未完成，调用该方法会阻塞直到任务完成。
6、带有超时的获取结果： get(long timeout, TimeUnit unit)方法可以设置超时时间，如果在指定时间内任务没有完成，则抛出TimeoutException。
实例

```
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class FutureTaskExample {
    public static void main(String[] args) {
        // 创建一个Callable任务
        Callable<String> callableTask = () -> {
            Thread.sleep(2000);
            return "Task completed";
        };

        // 创建FutureTask，将Callable任务包装起来
        FutureTask<String> futureTask = new FutureTask<>(callableTask);

        // 创建线程执行FutureTask
        Thread thread = new Thread(futureTask);
        thread.start();

        try {
            // 获取任务执行结果（阻塞等待任务完成）
            String result = futureTask.get();
            System.out.println(result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}

```

## 构造函数

- FutureTask(Callable<V> callable)

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

这个构造函数会把传入的Callable变量保存在this.callable字段中，该字段定义为`private Callable<V> callable`;用来保存底层的调用，在被执行完成以后会指向null,接着会初始化state字段为NEW。

- FutureTask(Runnable runnable, V result)

```java
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

这个构造函数会把传入的Runnable封装成一个Callable对象保存在callable字段中，同时如果任务执行成功的话就会返回传入的result。这种情况下如果不需要返回值的话可以传入一个null。

顺带看下Executors.callable()这个方法，这个方法的功能是把Runnable转换成Callable，代码如下:

```java
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
       throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
```

可以看到这里采用的是适配器模式，调用`RunnableAdapter<T>(task, result)`方法来适配，实现如下:

```java
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

这个适配器很简单，就是简单的实现了Callable接口，在call()实现中调用Runnable.run()方法，然后把传入的result作为任务的结果返回。

在new了一个FutureTask对象之后，接下来就是在另一个线程中执行这个Task,无论是通过直接new一个Thread还是通过线程池，执行的都是run()方法，接下来就看看run()方法的实现。

