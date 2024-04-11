# Java定时任务的使用方法
## JDK自带Timer实现
目前来看，JDK自带的Timer API算是最古老的定时任务实现方式了。Timer是一种定时器工具，用来在一个后台线程计划执行指定任务。它可以安排任务“执行一次”或者定期“执行多次”。
在实际的开发当中，经常需要一些周期性的操作，比如每5分钟执行某一操作等。对于这样的操作最方便、高效的实现方式就是使用java.util.Timer工具类。
### 核心方法
```
// 在指定延迟时间后执行指定的任务
schedule(TimerTask task,long delay);
// 在指定时间执行指定的任务。（只执行一次）
schedule(TimerTask task, Date time);
// 延迟指定时间（delay）之后，开始以指定的间隔（period）重复执行指定的任务
schedule(TimerTask task,long delay,long period);
// 在指定的时间开始按照指定的间隔（period）重复执行指定的任务
schedule(TimerTask task, Date firstTime , long period);
// 在指定的时间开始进行重复的固定速率执行任务
scheduleAtFixedRate(TimerTask task,Date firstTime,long period);
// 在指定的延迟后开始进行重复的固定速率执行任务
scheduleAtFixedRate(TimerTask task,long delay,long period);
// 终止此计时器，丢弃所有当前已安排的任务。
cancal()；
// 从此计时器的任务队列中移除所有已取消的任务。
purge()；
```
### 实例
定时器任务1
```
package Timer;

import java.util.Date;
import java.util.TimerTask;

public class JDKTimer extends TimerTask {
    private String taskName;
    public JDKTimer(String taskName) {
        this.taskName = taskName;
    }
    @Override
    public void run() {
        System.out.println(new Date() + " : 任务「" + taskName + "」被执行。");
    }
}
```
定时器任务2
```
package Timer;

import java.util.Date;
import java.util.TimerTask;

public class JDKTimer01 extends TimerTask {

    private String taskName;

    public JDKTimer01(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {
        System.out.println(new Date() + " : 任务「" + taskName + "」被执行。");
    }
}
```
主函数
```
package Timer;

import java.util.Timer;

public class Main {
    public static void main(String[] args) {
        Timer timer = new Timer();
        timer.scheduleAtFixedRate(new JDKTimer("timer01"),2000L,1000L);
        timer.scheduleAtFixedRate(new JDKTimer01("timer02"),3000L,2000L);
    }
}
```
schedule侧重保持间隔时间的稳定
scheduleAtFixedRate保持执行频率的稳定
### Timer的缺陷
Timer计时器可以定时（指定时间执行任务）、延迟（延迟5秒执行任务）、周期性地执行任务（每隔个1秒执行任务）。但是，Timer存在一些缺陷。首先Timer对调度的支持是基于绝对时间的，而不是相对时间，所以它对系统时间的改变非常敏感。
其次Timer线程是不会捕获异常的，如果TimerTask抛出的了未检查异常则会导致Timer线程终止，同时Timer也不会重新恢复线程的执行，它会错误的认为整个Timer线程都会取消。同时，已经被安排单尚未执行的TimerTask也不会再执行了，新的任务也不能被调度。故如果TimerTask抛出未检查的异常，Timer将会产生无法预料的行为。

## JDK自带ScheduledExecutorService
### 核心方法
```
ScheduledFuture<?> schedule(Runnable command,long delay, TimeUnit unit);
<V> ScheduledFuture<V> schedule(Callable<V> callable,long delay, TimeUnit unit);
ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnitunit);
ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnitunit);
```
其中scheduleAtFixedRate和scheduleWithFixedDelay在实现定时程序时比较方便，运用的也比较多。
ScheduledExecutorService中定义的这四个接口方法和Timer中对应的方法几乎一样，只不过Timer的scheduled方法需要在外部传入一个TimerTask的抽象任务。
而ScheduledExecutorService封装的更加细致了，传Runnable或Callable内部都会做一层封装，封装一个类似TimerTask的抽象任务类（ScheduledFutureTask）。然后传入线程池，启动线程去执行该任务。
### scheduleAtFixedRate方法
scheduleAtFixedRate方法，按指定频率周期执行某个任务。定义及参数说明：
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
				long initialDelay,
				long period,
				TimeUnit unit);
复制代码
参数对应含义：command为被执行的线程；initialDelay为初始化后延时执行时间；period为两次开始执行最小间隔时间；unit为计时单位。
使用实例：
public class ScheduleAtFixedRateDemo implements Runnable{

    public static void main(String[] args) {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
        executor.scheduleAtFixedRate(
                new ScheduleAtFixedRateDemo(),
                0,
                1000,
                TimeUnit.MILLISECONDS);
    }

    @Override
    public void run() {
        System.out.println(new Date() + " : 任务「ScheduleAtFixedRateDemo」被执行。");
        try {
            Thread.sleep(2000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
复制代码
上面是scheduleAtFixedRate方法的基本使用方式，但当执行程序时会发现它并不是间隔1秒执行的，而是间隔2秒执行。
这是因为，scheduleAtFixedRate是以period为间隔来执行任务的，如果任务执行时间小于period，则上次任务执行完成后会间隔period后再去执行下一次任务；但如果任务执行时间大于period，则上次任务执行完毕后会不间隔的立即开始下次任务。
### scheduleWithFixedDelay方法
scheduleWithFixedDelay方法，按指定频率间隔执行某个任务。定义及参数说明：
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
				long initialDelay,
				long delay,
				TimeUnit unit);
复制代码
参数对应含义：command为被执行的线程；initialDelay为初始化后延时执行时间；period为前一次执行结束到下一次执行开始的间隔时间（间隔执行延迟时间）；unit为计时单位。
使用实例：
public class ScheduleAtFixedRateDemo implements Runnable{

    public static void main(String[] args) {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
        executor.scheduleWithFixedDelay(
                new ScheduleAtFixedRateDemo(),
                0,
                1000,
                TimeUnit.MILLISECONDS);
    }

    @Override
    public void run() {
        System.out.println(new Date() + " : 任务「ScheduleAtFixedRateDemo」被执行。");
        try {
            Thread.sleep(2000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
复制代码
上面是scheduleWithFixedDelay方法的基本使用方式，但当执行程序时会发现它并不是间隔1秒执行的，而是间隔3秒。
这是因为scheduleWithFixedDelay是不管任务执行多久，都会等上一次任务执行完毕后再延迟delay后去执行下次任务。
## Quartz框架实现




