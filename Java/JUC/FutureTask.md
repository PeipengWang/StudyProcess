### FutureTask

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

