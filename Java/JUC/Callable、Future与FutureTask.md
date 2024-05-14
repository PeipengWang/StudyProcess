
### Callable、Future与FutureTask

参考：http://concurrent.redspider.group/article/01/2.html

们使用Runnable和Thread来创建一个新的线程。但是它们有一个弊端，就是run方法是没有返回值的。而有时候我们希望开启一个线程去执行一个任务，并且这个任务执行完成后有一个返回值。JDK提供了Callable接口与Future接口为我们解决这个问题，这也是所谓的“异步”模型。

使用Callable接口创建多线程

```
public class TaskCallAble implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        System.out.println(Thread.currentThread().getName());
        Thread.sleep(5000);
        return 111;
    }
}

```

注意Callable是一个带有泛型的接口，call返回的数据类型就是接口类型

```
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

通过线程池启动Callable接口的异步任务

```
  public static void main(String args[]) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        TaskCallAble taskCallAble = new TaskCallAble();
        //这里通过线程池提交任务
        Future<Integer> submit = executorService.submit(taskCallAble);
        System.out.println(submit.isDone());
        //这里获取异步任务的结果，会堵塞
        System.out.println(submit.get());
        System.out.println("aaa");
        System.out.println(submit.isDone());
    }
```

输出为：
false
pool-1-thread-1
111
aaa
true

提交任务后立即开始执行，执行过程中可以用Future接口来对线程进行操作
其中Future接口有几个参数

```
public abstract interface Future<V> {
    //试图取消任务
    public abstract boolean cancel(boolean paramBoolean);
    //查看是否取消
    public abstract boolean isCancelled();
    //查看任务是否执行完毕
    public abstract boolean isDone();
    //获取线程执行结果
    public abstract V get() throws InterruptedException, ExecutionException;
    public abstract V get(long paramLong, TimeUnit paramTimeUnit)
            throws InterruptedException, ExecutionException, TimeoutException;
}
```

当然我们还有一种更加灵活的异步执行方法--FutureTask
FutureTask是Java中Future接口的一个具体实现，它提供了一种异步计算的机制，允许你在一个线程中执行某个任务，并在另一个线程中获取该任务的执行结果。
上面利用FutureTask进行包装，可以直接利用Thread()来获取异步结果

```
FutureTask<Integer> integerFutureTask = new FutureTask<>(new TaskCallAble());
Thread thread = new Thread(integerFutureTask);
thread.start();
System.out.println(integerFutureTask.get());
```

在这个示例中，我们手动创建了一个FutureTask对象，并将其包装在一个Thread对象中启动执行。主线程在启动任务后继续执行其他操作，而不需要等待任务完成。这样可以提高程序的并发性，充分利用系统资源。
执行结果为：

```
Thread-0
111
```
