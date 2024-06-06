## 第一章
### 多线程创建的三种方式
创建第一个多线程实例（继承Thread）
```
public class MyThread extends Thread{
    @Override
    public void run(){
        super.run();
        System.out.println("MyThread");
    }
}
```
实际为继承Thread，重写run方法
调用方法为
```
MyThread myThread1 = new MyThread();
myThread1.start();
```
通过start()方法创建一个新线程，但是需要注意的是，start方法是将线程放入就绪态，并不是立即执行。
另外两种创建多线程方法
1、实现Runnable接口
```
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        try {
            System.out.println("MyRunnable:"+Thread.currentThread());
            Thread.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
传入Thread，并使用start启动
```
Runnable runnable = new MyRunnable();
Thread thread2 = new Thread(runnable);
thread2.start();
```
优势：
Thread是继承的方式实现的，但是Java是单继承，不支持多继承，导致MyThread类不能继承其他的父类，而通过Runable接口可以间接实现多继承。
2、实现Callable接口

### 变量之间的共享问题
定义一个count变量，初始值设为5，每次减1，循环显示数值，直到0
```
public class MyThread extends Thread{
    private String name;
    private int count = 5;
    public MyThread(String name){
        this.name = name;
    }
    @Override
    public void run(){
        super.run();
        try {
            while (count>0){
                System.out.println(name+":"+count--);
                Thread.sleep(300);
            }

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
定义线程
```
  MyThread myThread1 = new MyThread("A");
  MyThread myThread2 = new MyThread("B");
  myThread2.start();
  myThread1.start();
```
显示结果为：
B:5
A:5
B:4
A:4
B:3
A:3
B:2
A:2
B:1
A:1
会发现两个线程各自显示自己的，不会共享
那么怎样使得数据共享呢，那就要借助static关键字了，定义count如下所示
```
private static int count = 5;
```
显示结果为：
B:5
A:4
B:3
A:2
B:1
但是这个时候会出现问题--**线程安全问题**
接下来将会展示这个问题，典型的抢票问题，count为总票数，A，B两个用户抢票，存在则减一
例如，在定义的线程中
```
public class MyThread extends Thread{
    private String name;
    private static int count = 500;
    public MyThread(String name){
        this.name = name;
    }
    @Override
    public void run(){
        super.run();
        while (count>0){
            System.out.println(name+":"+count--);
        }
    }
}

```
这里只罗列最后的结果
B:6
B:5
B:4
B:3
B:2
B:1
A:235
虽然B用户已经将票抢完了，但是A明显没反应过来，记录的依然是235张票，因此为例线程安全，可以使用sychronezed函数。
```
public class MyThread extends Thread{
    private String name;
    private static int count = 500;
    public MyThread(String name){
        this.name = name;
    }
    @Override
     public void run(){
        while (count>0){
            try {
                Thread.sleep(1);
                synchronized (MyThread.class){
                    //为了防止拿到锁之后票还是没了
                    if (count > 0){
                        System.out.println(name+":"+count--);
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }
}

```
在这里，通过添加类锁的方式对静态变量加锁，这样就可以解决这个问题了。在这里进行初步讨论，后面深入分析。
### 线程常用方法
1、isAlive()判断当前线程是否存活，如果线程处在正在运行和准备运行的状态为活动状态。  
2、sleep(long millis) 在指定时间内让当前“正在执行的线程”休眠，sleep状态下的线程为存活的。  
3、StackTraceElement[] getStackTrace() 方法，返回一个表示改线程堆栈跟踪元素数组。  
4、getId() 取得线程的唯一标识  
5、停止线程，不建议使用Thread.stop()已废弃，建议使用Thread.interrupt()，仅仅是在当前线程中做了一个停止标记，并不是真正停止线程。  
停止线程有interrupt+抛异常法和interrupt+return两种比较好的方法  
isInterrupted()能判断线程是否中断  
6、暂停线程suspend()，恢复线程resume()  
如果线程在获取了锁之后，被suspend()，容易导致这个线程独占锁，产生异常  
也容易出现线程暂停，从而导致数据不完整。因此这是个过期方法。  
7、yield()方法，放弃当前CPU资源，让其他任务去占用CPU执行时间，放弃的时间不确定，有可能刚刚放弃，又马上获得时间片  
8、setPriority(int Priority)，设置 线程优先级，范围为1-10  
### 守护线程
守护线程是一种特殊的线程，当进程中不存在非守护线程了，则守护线程自动销毁，GC就是一种典型的守护线程。  
setDaemon(true)即将线程设置为守护线程。  
例如  
设置一个每秒打印五个数字的线程  
```
public class MyRunnable implements Runnable{

    @Override
    public void run() {
        try {
            System.out.println("MyRunnable:"+Thread.currentThread());
            Thread.sleep(300);
            int i = 0;
            while (true){
                System.out.println(i);
                i++;
                Thread.sleep(200);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
在main函数中调用，设置上述线程为守护线程，主线程main为用户线程，当用户线程结束，守护线程也会结束  
```
  Runnable runnable = new MyRunnable();
  Thread thread2 = new Thread(runnable);
  //先设置再启动
  thread2.setDaemon(true);
  thread2.start();
  Thread.sleep(2000);
  System.out.println("主线程结束了");
```
打印结果为：  
MyRunnable:Thread[Thread-0,5,main]  
0
1
2
3
4
5
6
7
8
主线程结束了  
## 第二章 对象及变量的并发访问
非线程安全问题存在于实例变量中，对于方法内部的私有变量，则不存在非线程安全问题。  
如果多个线程访问一个对象中的实例变量，则有可能出现线程安全的问题。  
或者说多个线程同时访问同一对象中没有同步的方法，会出现线程安全问题。  
需要牢记：只有多个线程“共享”的资源才需要进行同步，没有“共享”就没必要同步。  
### synchronized同步方法与同步代码块  
关键字sychronized可用来保障原子性、可见性和有序性  
实例查看第一章第二节  
1、原理  
方法同步的原理：sychronized能实现同步的原因是使用了flag标记ACC_SYCHRONIZED，当调用方法时，调用指令会查看ACC_SYCHRINIZED访问标志，如果设置了，执行线程先持有同步锁，然后执行方法，最后执行完成后释放同步锁。  
代码块同步的原理：如果使用sychronized同步代码块，则使用monitorenter和monitorexit指令进行了同步处理  
### synchronized同步对象
锁对象有多种表现形式  
1、锁住所有对象的方法，例如下面的MyObject对象，methodA与methodB是同步的，调用methodA就不会调用methodB  
```
public class MyObject {
    public synchronized void methodA() throws InterruptedException {
        System.out.println("start methodA");
        Thread.sleep(1000);
        System.out.println("end methodA");
    }
    public synchronized void methodB() throws InterruptedException {
        System.out.println("start methodB");
        Thread.sleep(1000);
        System.out.println("end methodB");
    }
}
```
需要知道：在方法声明中添加sychronized并不是锁方法，而是锁 当前类的对象。在Java中，只有将“当前对象作为锁”这中说法，并没有“锁方法”这种说法。  
sychronized锁重入  
可重入锁是指自己可以再次获取自己的内部锁，此时这个锁还没有释放，当其再次想要获取这个对象锁时还是可以再此获取的。  
锁重入也支持父子类继承环境。  
sychronized对象方法继承重写  
子类中重写父类方法，如果不加sychronized，会变为非同步方法  
### synchronized同步代码块  
同步代码块为在方法中使用  
```
sychronized(this){
     业务代码；
}
```
这个会持有当前所在类的对象锁。  
若是使用非this对象x的同步代码块格式进行操作时，锁必须是同一个。  
例如  
```
sychronized(x){
     业务代码；
}
```
这个代码块锁住的是x这个对象，而不是所在类，如果多个线程传入的是一个x，那业务代码则同步，如果不是同一个对象，则业务代码不同步。  
首先建立一个对象，这个对象主要用来加锁验证  
```
public class MyObject {
}

``` 
然后建立一个服务，在这个服务中不是对this加锁，而是对object加锁  
```
public class Service {
    public void service(MyObject object) throws InterruptedException {
        synchronized (object){
            System.out.println("enter methods"+Thread.currentThread().getName());
            Thread.sleep(1000);
            System.out.println("exit methods"+Thread.currentThread().getName());
        }
    }
}

```
分别建立两个线程，调用service，同时将object传给service方法  
```
public class MyThread1 extends Thread{
    private MyObject myObject;
    private Service service;
    public MyThread1(MyObject object, Service service){
        this.myObject = object;
        this.service = service;
    }
    @Override
     public void run(){
        try {
            service.service(myObject);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```
public class MyThread2 extends Thread{
    private MyObject myObject;
    private Service service;
    public MyThread2(MyObject object, Service service){
        this.myObject = object;
        this.service = service;
    }
    @Override
     public void run(){
        try {
            service.service(myObject);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
main函数执行方法  
```
        MyObject object = new MyObject();
        Service service = new Service();
        MyThread1 thread1 = new MyThread1(object,service);
        thread1.setName("A");
        MyThread2 thread2 = new MyThread2(object, service);
        thread2.setName("B");
        thread1.start();
        thread2.start();
```
这里object是同一个对象  
执行流程  
enter methodsA  
exit methodsA  
enter methodsB  
exit methodsB  
可以看出代码块是同步的。  
但是，当object不是同一个对象时，就无法同步了  
修改代码为  
```
        MyObject object1 = new MyObject();
        MyObject object2 = new MyObject();
        Service service = new Service();
        MyThread1 thread1 = new MyThread1(object1,service);
        thread1.setName("A");
        MyThread2 thread2 = new MyThread2(object2, service);
        thread2.setName("B");
        thread1.start();
        thread2.start();
```
执行结果  
enter methodsA  
enter methodsB  
exit methodsA  
exit methodsB  
可以看出这是不同步的，因为在service中锁住的是object对象，这个对象不是一个的时候不会同步。  
```
 synchronized (object){
            System.out.println("enter methods"+Thread.currentThread().getName());
            Thread.sleep(1000);
            System.out.println("exit methods"+Thread.currentThread().getName());
        }
```
### synchronized同步静态方法  
关键字sychronized还可以应用在static静态方法和静态代码块上，这样写是对对应的java文件的Class类持有锁，Class类是单例的，更具体的说，在静态static方法上使用sychronized关键字声明同步方法时，使用当前静态方法所在的类对应Class类的单例对象作为锁  
对象锁针对的是同一个对象，如果是两个对象的话，就无法同步了，但是类锁针对的是整个类。需要注意的是两者之间是并列关系，而不是包含关系，换句话说，对象锁不能与类锁同步，对象锁只能与对象锁同步，类锁也只能与类锁同步。  
下面有个实例可以说明这一点：  
建立一个包含类锁和对象锁的类  
```
public class Service {
    //类锁
    synchronized public static void methodA() throws InterruptedException {
        System.out.println("enter methodA"+Thread.currentThread().getName());
        Thread.sleep(1000);
        System.out.println("exit methodA"+Thread.currentThread().getName());
    }
    //类锁
    synchronized public static void methodB() throws InterruptedException {
        System.out.println("enter methodB"+Thread.currentThread().getName());
        Thread.sleep(1000);
        System.out.println("exit methodB"+Thread.currentThread().getName());
    }
    //对象锁
    synchronized public  void methodC() throws InterruptedException {
        System.out.println("enter methodC"+Thread.currentThread().getName());
        Thread.sleep(1000);
        System.out.println("exit methodC"+Thread.currentThread().getName());
    }
}
```
建立三个线程，分别执行上述methodA，methodB与methodC  
```
public class MyThread1 extends Thread{
    private MyObject myObject;
    private Service service;
    public MyThread1(MyObject object, Service service){
        this.myObject = object;
        this.service = service;
    }
    @Override
     public void run(){
        try {
            Service.methodA();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
```
public class MyThread2 extends Thread{
    private MyObject myObject;
    private Service service;
    public MyThread2(MyObject object, Service service){
        this.myObject = object;
        this.service = service;
    }
    @Override
     public void run(){
        try {
            Service.methodB();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
```
public class MyThread3 extends Thread{
    private MyObject myObject;
    private Service service;
    public MyThread3(MyObject object, Service service){
        this.myObject = object;
        this.service = service;
    }
    @Override
     public void run(){
        try {
            service.methodC();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
对每一个线程建立一个新的Service来执行线程。  
```
        MyObject object1 = new MyObject();
        Service service1 = new Service();
        MyThread1 thread1 = new MyThread1(object1,service1);
        thread1.setName("1");
        Service service2 = new Service();
        MyThread2 thread2 = new MyThread2(object1, service2);
        thread2.setName("2");
        Service service3 = new Service();
        MyThread3 thread3 = new MyThread3(object1,service3);
        thread3.setName("3");
        thread1.start();
        thread2.start();
        thread3.start();
```
enter methodA1  
enter methodC3  
exit methodA1  
enter methodB2  
exit methodC3  
exit methodB2  
可以看出，A1与B2是同步的，但是C3与前两者都不同步。  
可知：类锁与对象锁不会同步。  
### synchronized同步静态代码块  
synchronized同步静态代码块跟同步对象代码块有写法上的区别  

```
synchronized(this.class){
    业务代码；
}
```
### synchronized使用需要注意的几个问题  
1、同步常量池元素，例如sychronized(String元素)  
String a = "1";  
String b = "1";  
在此代码中a与b指向的元素是同一个，都是指向常量池中的“1”，因此，如果对a加锁实际上也是对b加了锁，这在有些情况下可能不符合实际情况，因此如果sychronized代码块不能使用String作为锁对象，它并不放在缓存池里，或者执行new String()创建不同的字符串对象，形成不同的锁。  
2、同步synchronized方法无限等待问题与解决方案  
更改同步方法为同步块，并对不同的对象加锁。  
3、多线程死锁  
可以看一个死锁的实例  
```

public class DeadThread implements Runnable{
    public String username;
    public Object lock1 = new Object();
    public Object lock2 = new Object();
    public void setFlagName(String name){
        this.username = name;
    }
    @Override
    public void run() {
        if(username.equals("a")){
            synchronized (lock1){
                System.out.println("enter "+username);
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (lock2){
                    System.out.println("exit "+username);
                }
            }
        }
        if(username.equals("b")){
            synchronized (lock2){
                System.out.println("enter "+username);
                try {
                    //
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (lock1){
                    System.out.println("exit "+username);
                }
            }
        }
    }
}

```
```
        DeadThread dead1 = new DeadThread();
        dead1.setFlagName("a");
        Thread thread1 = new Thread(dead1);
        thread1.start();
        Thread.sleep(1000);
        dead1.setFlagName("b");
        Thread thread2 = new Thread(dead1);
        thread2.start();

```
在这个线程 中，定义lock1余lock2两个对象锁，存在互相持有对方锁的情况  
用DeadThread 开辟两个Thread线程，这时会进入死锁状态，线程a持有lock1锁后等待3s，主线程同时在1s秒后创建线程b，线程b持有lock2锁等待2秒。在2s秒，线程a试图获取lock2的锁，线程b试图获取lock1的锁，但是两个锁都分别被对方线程持有，这种互相持有对方锁的方式导致两个线程卡主成为死锁。  
可以通过JDK自带工具来监测是否有死锁现象  
jps查看进程，jstack -l 进程 来查看线程运行情况  
标志为waiting to lock  
4、锁对象改变也会导致一些错误  
通常情况下，一旦持有锁后就不再对对象进行更改，因为一旦更改可能导致其他线程持有的锁对象发生改变，由同步变为异步。  

### volatile关键字  
特性：  
1、可见性：B线程能马上看到A线程修改的数据  
2、原子性  
3、禁止代码重排序  
#### 可见性测试  
首先来看一下单线程与多线程是怎样出现死循环的。  
简单设定一个打印函数类 public void printMethod() 通过这个方法来打印日志，知道设置的flag值为false时退出打印  
```
public class PrintString {
    private boolean flag = true;

    public void setFlag(boolean flag) {
        this.flag = flag;
    }
    public void printMethod() throws InterruptedException {
        while (flag){
            System.out.println("进入了方法"+Thread.currentThread().getName());
            Thread.sleep(1000);
        }
    }
}
```
主函数执行  
```
        PrintString printString = new PrintString();
        printString.printMethod();
        Thread.sleep(2000);
        printString.setFlag(false);
```
单线程情况下很容易理解，即代码无法走到printString.setFlag(false);这一步，整个线程卡住。  
可以用多线程的方式来解决，如下所示  
```
public class PrintString implements Runnable{
    private Boolean flag = true;

    public void setFlag(Boolean flag) {
        this.flag = flag;
    }
    public void printMethod() throws InterruptedException {
        while (flag){
            System.out.println("进入了方法"+Thread.currentThread().getName());
            Thread.sleep(1000);
        }
    }

    @Override
    public void run() {
        try {
            printMethod();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
这个是可以正常中断的，但是有一种极端情形，在while(flag)中什么都不执行  
```
package com.Multithreading;

public class PrintString implements Runnable{
    private Boolean flag = true;

    public void setFlag(Boolean flag) {
        this.flag = flag;
    }
    public void printMethod() throws InterruptedException {
        while (flag){
        //什么都没有
        }
        System.out.println("exit");
    }

    @Override
    public void run() {
        try {
            printMethod();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
这种情况下下即使执行如下代码，也会卡死。  
```
        PrintString print1 = new PrintString();
        Thread thread1 = new Thread(print1);
        thread1.start();
        Thread.sleep(2000);
        print1.setFlag(false);
        System.out.println("end main");
```
为什么会出现这种现象呢？这就涉及到JMM相关了  
变量private Boolean flag = true;存在于公共堆栈（共享内存），线程运行后一直在线程私有栈中取得 flag = true，而代码 print1.setFlag(false);虽然被执行，但是更新的却是公共堆栈的信息，所以线程一直处在死循环状态。  
在这篇博客中[JMM](http://toutiao.com/article/6893431027092423180/?wid=1660807363699)对于这个问题有所回答：  
假设主内存中存在一个共享变量 x ，现在有 A 和 B 两个线程分别对该变量 x=1 进行操作， A/B线程各自的工作内存中存在共享变量副本 x 。假设现在 A 线程想要修改 x 的值为 2，而 B 线程却想要读取 x 的值，那么 B 线程读取到的值是 A 线程更新后的值 2 还是更新钱的值 1 呢？  

答案是：不确定。即 B 线程有可能读取到 A 线程更新钱的值 1，也有可能读取到 A 线程更新后的值 2，这是因为工作内存是每个线程私有的数据区域，而线程 A 操作变量 x 时，首先是将变量从主内存拷贝到 A 线程的工作内存中，然后对变量进行操作，操作完成后再将变量 x 写回主内存。而对于 B 线程的也是类似的，这样就有可能造成主内存与工作内存间数据存在一致性问题，假设直接的工作内存中，这样 B 线程读取到的值就是 x=1 ，但是如果 A 线程已将 x=2 写回主内存后，B线程才开始读取的话，那么此时 B 线程读取到的就是 x=2 ，但到达是那种情况先发送呢？  

在设置的如下代码中，不断读取flag的值，导致写入实际上是失效的。那么回归刚开始能够解决问题的代码中，如果在特殊情况下，写入数据没有成功，多线程是不是也会出现问题呢。  
```
  while (flag){
        //什么都没有
        }
```
因此，volitaile关键字是解决这个问题的利器。  
因为它直接在公共堆栈中读取和更新，他对于所有线程是可见的。  
修改变量为  
```
private volatile Boolean flag = true;
```
执行main函数  
```
        PrintString print1 = new PrintString();
        Thread thread1 = new Thread(print1);
        thread1.start();
        Thread.sleep(2000);
        print1.setFlag(false);
        System.out.println("end main");
```
end main  
exit  

Process finished with exit code 0  
能够正常的退出。  

## 线程间的通信  
### wait/notify机制  
wait()方法是object()类的方法，它的作用是使当前执行的wait()方法的线程等待，在wait()方法所在的代码处暂停，并释放锁，直到接到通知被中断为止。在调用wait()方法之前，线程必须获得改对象的对象级别的锁，即只能在同步方法和同步块中调用wait()方法。通过通知机制使得某个线程继续执行wait()方法后面的代码时，对线程的选择是按照执行wait()方法的顺序决定的，并需要重新获得锁。如果没有获得锁则抛出异常。  
notify()方法是要在同步方法或同步块中调用，在调用前，线程必须获得锁，如果调用线程没有获得适当的锁，则会抛出异常。该方法用来通知那些可能等待锁的其他线程，如果有多个线程等待，则会按照执行wait()方法的顺序对处于wait()状态的线程发出一次通知，并使该线程重新获得锁。需要注意的是执行notify()方法后，当前线程不会马上释放改锁，呈wait()状态的线程也不可能马上获取改对象锁，要等到执行notify()方法的线程将程序执行完，也就是退出sychronized同步区域后，当前线程才会释放锁，而呈wait状态的线程才会获取改对象锁。当第一个获得了该对象的wait()线程运行完毕后，它会释放该对象锁，此时如果没有再使用notify()语句，那么其他呈wait()对象的线程因为没有获得通知，会持续处于wait()状态。  

### wait()/notify()基本使用  
1、不加锁会抛出异常IllegalMonitorStateException  
```
public class Main {
    public static void main(String[] args) throws InterruptedException {
        String str = new String("aaa");
        System.out.println("wait before");
        str.wait();
        System.out.println("wait after");
    }
}

wait before
Exception in thread "main" java.lang.IllegalMonitorStateException
    at java.lang.Object.wait(Native Method)
    at java.lang.Object.wait(Object.java:502)
    at com.Multithreading.waitAndNotify.Main.main(Main.java:7)

```
加上synchronized (str)才可以
```
public class Main {
    public static void main(String[] args) throws InterruptedException {
        String str = new String("aaa");
        System.out.println("wait before");
        synchronized (str){
            str.wait();
        }
        System.out.println("wait after");
    }
}
```
2、notify唤醒wait操作  
建立一个执行wait的线程  
```
public class MyThread1 extends Thread{
    private Object lock;
    public MyThread1(Object lock){
        super();
        this.lock = lock;
    }
    @Override
    public void run() {
        super.run();
        synchronized (lock){
            System.out.println("wait start");
            try {
                Thread.sleep(1000);
                lock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("wait end");
        }
    }
}

```
主线程执行notify()操作  
```
  Object lock = new Object();
       MyThread1 thread1 = new MyThread1(lock);
       thread1.start();
       Thread.sleep(1200);
       System.out.println("notify start");
       synchronized (lock){
           lock.notify();
           Thread.sleep(1000);
           System.out.println("notify end");
       }
    }
```
结果为：  
```
wait start
notify start
notify end
wait end
```
可以看出notify唤醒了wait线程，但是是在执行完notify线程后，wait线程才会重新获取。  
3、wait方法与sleep方法的区别  
执行wait方法锁会被立即释放  
执行sleep方法会持续持有锁  
notify()方法也不会释放锁  
在执行代码过程中，代码异常也会自动释放锁  
4、notifyAll()方法会唤醒所有wait()线程  
唤醒的顺序是正序、倒序、随机，取决于JVM的实现。 
5、wait(long) 方法  
wait(long)方法功能是等待某一时间内，如果超过这个时间则自动唤醒，如果这段时间有线程对锁进行notify()则立即唤醒。  
方法想要自动向下也需要重新获得锁。如果没有锁会一直等待，直到持有锁为止。  
### 生产者/消费者模式的实现  
### 通过管道进行线程间的通信  
### join()  
1、join()方法的作用是使所属的线程对象x正常执行run()方法中的任务，而使当前线程z进行无限期的阻塞，等待线程x销毁后再继续执行线程z后面的代码，具有串联效果。  
join()方法具有使线程排队运行的效果，有些类似同步的运行效果，但是join()方法与sychronized的区别是join()方法在内部使用wait()方法进行等待，而sychronized关键字使用锁作为同步。  
2、x.join(long)方法为不管x线程是否结束，只要时间到了立即尝试获得锁，继续向后执行。  
3、join方法与sleep方法区别：  
join方法内部使用wait方法实现的，具有释放锁的效果，sleep方法会持续持有锁。  
4、一个意外，待看  
p270
### ThreadLocal  
public static 的变量值在所有的线程是共享的，JDK提供的ThreadLocal可以使得每一个线程都有自己的变量。  
类ThreadLocal的主要作用是将数据放入当前线程的Map中，这个Map是Thread类的实例变量。类ThreadLocal自己不管理、不存储任何数据，它只是数据和Map之间的桥梁，用于将数据放入Map中，执行流程如下：数据-》ThreadLocal-》currentThread()-》Map。
执行后每个线程中的map都有自己的数据，Map中的key存储的是ThreadLocal对象，value就是存储的值。每个Thread中的Map值只对当前线程可见，其他线程不可以访问当前线程中Map值。当前线程销毁，Map随之销毁，Map中的数据如果没有被引用、没有被使用，则随时GC收回。  

ThreadLocal()是不能实现继承的，InheritableThreadLocal可以使得子线程继承父线程的值。  
## Local对象的使用  
### ReetrantLock类  
基本使用  
执行服务  
```
public class MyService {
    private Lock lock = new ReentrantLock();
    public void testMethod(){
        try{
            lock.lock();
            for(int i = 0; i <5; i++){
                System.out.println(Thread.currentThread().getName()+"-"+i);
            }
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }


    }
}
```
建立线程  
```
public class MyThread extends Thread{
    private MyService service;
    public MyThread (MyService service){
        this.service = service;
    }
    @Override
    public void run(){
        service.testMethod();
    }
}
```
新建线程执行测试  
```
public class Run {
    public static void main(String[] args) {
        MyService service = new MyService();
        MyThread thread1 = new MyThread(service);
        MyThread thread2 = new MyThread(service);
        MyThread thread3 = new MyThread(service);
        MyThread thread4 = new MyThread(service);
        MyThread thread5 = new MyThread(service);
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();

    }
}
```
结果  
```
Thread-0-0  
Thread-0-1
Thread-0-2
Thread-0-3
Thread-0-4
Thread-2-0
Thread-2-1
Thread-2-2
Thread-2-3
Thread-2-4
Thread-3-0
Thread-3-1
Thread-3-2
Thread-3-3
Thread-3-4
Thread-1-0
Thread-1-1
Thread-1-2
Thread-1-3
Thread-1-4
Thread-4-0
Thread-4-1
Thread-4-2
Thread-4-3
Thread-4-4
```
可以看到每个线程在执行Myservice的服务层方法时是依次输出的  
### await()/signal()方法  
await()方法是与wait()方法类似的一个方法，算是wait()方法的加强版。  
wait/notify机制中，等待的线程被通知时是由JVM进行选择，notifyAll()是通知所有waitting进程，用户是没有选择权的，await()可以进行选择性的通知。  
await()是借助Condition对象来实现多路选择通知的，可以创建多个Condition实例，从而有选择性的进行通知。  
Condition对象的作用是控制并处理线程的状态，它可以使线程呈wait状态，也可以让线程继续运行  
实例  
定义服务：  
```
public class MyService {
    private Lock lock = new ReentrantLock();
    Condition condition1 =lock.newCondition();
    public void await(){
        try{
            lock.lock();
            System.out.println("into wait: "+Thread.currentThread().getName());
            condition1.await();
            System.out.println("exit wait: "+Thread.currentThread().getName());
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void signal(){
        try {
            lock.lock();
            System.out.println("into signal: "+ Thread.currentThread().getName());
            condition1.signal();
            System.out.println("exit signal: "+ Thread.currentThread().getName());
            System.out.println();
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```
定义线程  
```
public class MyThread extends Thread{
    private MyService service;
    public MyThread (MyService service){
        this.service = service;
    }
    @Override
    public void run(){
        service.await();
    }
}

```
执行服务  
```
public class Run {
    public static void main(String[] args) throws InterruptedException {
        MyService service = new MyService();
        MyThread thread1 = new MyThread(service);
        MyThread thread2 = new MyThread(service);
        MyThread thread3 = new MyThread(service);
        MyThread thread4 = new MyThread(service);
        MyThread thread5 = new MyThread(service);
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();
        Thread.sleep(1000);
        service.signal();
        service.signal();
        service.signal();

    }
}
```
结果：  
into wait: Thread-0  
into wait: Thread-1    
into wait: Thread-2
into wait: Thread-3  
into wait: Thread-4  
into signal: main  
exit signal: main  

into signal: main  
exit signal: main 

into signal: main  
exit signal: main  

exit wait: Thread-0  
exit wait: Thread-1  
exit wait: Thread-2  
可以看到定义了四个线程，在主线程中，通知了三个线程，还有一个依然处于阻塞状态。  
也可以在服务中唤醒所有线程  
```
    public void signalAll(){
        try {
            lock.lock();
            System.out.println("into signalAll: "+ Thread.currentThread().getName());
            condition1.signalAll();
            System.out.println("exit signalAll: "+ Thread.currentThread().getName());
            System.out.println();
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```
调用  
```public class Run {
    public static void main(String[] args) throws InterruptedException {
        MyService service = new MyService();
        MyThread thread1 = new MyThread(service);
        MyThread thread2 = new MyThread(service);
        MyThread thread3 = new MyThread(service);
        MyThread thread4 = new MyThread(service);
        MyThread thread5 = new MyThread(service);
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();
        Thread.sleep(1000);
        service.signalAll();

    }
}

```
结果：  
into wait: Thread-0  
into wait: Thread-3  
into wait: Thread-1  
into wait: Thread-4  
into wait: Thread-2  
into signalAll: main  
exit signalAll: main  

exit wait: Thread-0  
exit wait: Thread-3  
exit wait: Thread-1  
exit wait: Thread-4  
exit wait: Thread-2  
唤醒所有线程，可以知道先等待的线程是先唤醒的。  

那么怎样唤醒指定的线程呢？  
在上述测试中定义了一个service，service中共用一个Condition对象，所以加的wait是一样的，要想单独唤醒需要定义多个Condition对象。  
重新定义服务  
```
public class MyService {
    private Lock lock = new ReentrantLock();
    Condition conditionA =lock.newCondition();
    Condition conditionB = lock.newCondition();
    public void awaitA(){
        try{
            lock.lock();
            System.out.println("into waitA: "+Thread.currentThread().getName());
            conditionA.await();
            System.out.println("exit waitA: "+Thread.currentThread().getName());
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void signalA(){
        try {
            lock.lock();
            System.out.println("into signalA: "+ Thread.currentThread().getName());
            conditionA.signal();
            System.out.println("exit signalA: "+ Thread.currentThread().getName());
            System.out.println();
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void awaitB(){
        try{
            lock.lock();
            System.out.println("into waitB: "+Thread.currentThread().getName());
            conditionB.await();
            System.out.println("exit waitB: "+Thread.currentThread().getName());
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void signalB(){
        try {
            lock.lock();
            System.out.println("into signalB: "+ Thread.currentThread().getName());
            conditionB.signal();
            System.out.println("exit signalB: "+ Thread.currentThread().getName());
            System.out.println();
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void signalAll(){
        try {
            lock.lock();
            System.out.println("into signalAll: "+ Thread.currentThread().getName());
            conditionA.signalAll();
            System.out.println("exit signalAll: "+ Thread.currentThread().getName());
            System.out.println();
            lock.unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
定义两个线程类分别执行awaitA和awaitB  
```
public class MyThreadA extends Thread{
    private MyService service;
    public MyThreadA (MyService service){
        this.service = service;
    }
    @Override
    public void run(){
        service.awaitA();
    }
}
```
```
public class MyThreadB extends Thread{
    private MyService service;
    public MyThreadB(MyService service){
        this.service = service;
    }
    @Override
    public void run(){
        service.awaitB();
    }
}
```  
执行主线程调用，在方法中定义五个线程，其中三个A两个B，最后通知A不通知B  
```
        MyService service = new MyService();
        MyThreadA thread1 = new MyThreadA(service);
        MyThreadA thread2 = new MyThreadA(service);
        MyThreadA thread3 = new MyThreadA(service);
        MyThreadB thread4 = new MyThreadB(service);
        MyThreadB thread5 = new MyThreadB(service);
        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
        thread5.start();
        Thread.sleep(1000);
        service.signalB();
        service.signalB();
```
执行结果  
into waitA: Thread-0  
into waitA: Thread-1  
into waitA: Thread-2  
into waitB: Thread-3  
into waitB: Thread-4  
into signalB: main  
exit signalB: main  

into signalB: main  
exit signalB: main  

exit waitB: Thread-3  
exit waitB: Thread-4  
这样实现了指定通知线程的功能了，但是需要注意的是指定通知只能指定不同Condition对象的锁，对于同一个Condition对象进行signal时会根据加锁的先后顺序来通知。  
### 生产者/消费者模式  
### 几个重要的方法  
1、public int getHoldCount()  
查询当前线程保持此锁定的的个数，即调用lock()方法的次数，执行lock()方法进行锁重入导致count()技术加一，执行unlock()方法会使得count减1.  
2、getQueueLength()  
返回正在等待获取此锁线程数的计数。  
3、getWaitQueueLength(Condition condition)  
返回等待此锁相关给定条件的线程技术，给定条件为condition对象。  
4、hasQueuedThread(Thread thread)  
查询指定线程是否等待获取此锁  
5、hasQueuedThreads()  
查询是否有线程正在等待获取此锁  
6、hasWaiters(Condition condition)  
查询是否有线程正在等待与此锁相关的condition条件  
7、isFair()  
是否是公平锁  
公平锁与非公平锁概念  

ReenTrantLock默认是非公平锁  
ReenTrantLock lock = new ReenTrantLock(true)；定义为公平锁  
ReenTrantLock lock = new ReenTrantLock(false)；定义为非公平锁  
8、isHeldByCurrentThread()  
查询当前线程是否保持此锁  
9、isLocked()  
查看当前线程是否由任意线程保持  
10、lockInterruptibly()  
当某个线程尝试获得锁并且阻塞在lockInterruptible方法时，该线程可以中断  
11、tryLock()  
嗅探拿锁，如果当前线程发现锁被其他线程持有，则返回false，程序继续执行后面的代码  
12、tryLock(long timeout,TimeUnit unit)  
嗅探拿锁，如果当前线程被其他线程占有了，如果在指定时间内持有了锁，则返回ture，否则则返回false，继续执行后面代码  
13、awaitNanos(long nanosTimeout)  
与await作用类似，有自动唤醒功能，单位为ns  
14、awaitUnitl(Date deadline)  
在固定的日期结束等待  
15、awitUninterruptible()  
在等待过程中不允许被中断  
### ReentrantReadWriteLock类  
ReentrantLock类具有完全排他互斥的效果，一个时间线只有一个线程在执行ReentrantLock.lock()后面的任务，这样做虽然保证了同时写实例变量的线程安全性，但是效率非常是非常低下的，所以JDK提供了一种读写锁ReentrantReadWriteLock类，使用他不需要同步执行，提升运行速度，加快运行效率。  
读写锁有两个锁：一个是读锁，也叫共享锁，一个是写锁，也叫排他锁。  
读锁之间不互斥，写锁与读锁互斥，写锁与写锁也互斥。  
实例：  
写写互斥实例  
服务层，写一个写锁  
lock.writeLock()  
```
public class MyService1 {
    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    private long start = System.currentTimeMillis();
    public void awaitA(){
        try{
            lock.writeLock().lock();
            System.out.println("into waitA: "+Thread.currentThread().getName());
            Thread.sleep(4000);
            long end = System.currentTimeMillis();
            System.out.println("exit waitA: "+Thread.currentThread().getName());
            System.out.println("wast time:"+ (end-start));
            lock.writeLock().unlock();
        }catch (Exception e){
            e.printStackTrace();
        }
    }

}
```
定义一个Thread  
```

public class MyThread extends Thread{
    private MyService1 service;
    public MyThread(MyService1 service){
        this.service = service;
    }
    @Override
    public void run(){
        service.awaitA();
    }
}

```
开辟两个线程  
```
public class Run2 {
    public static void main(String[] args) {
        MyService1 service1 = new MyService1();
        MyThread thread1 = new MyThread(service1);
        MyThread thread2 = new MyThread(service1);
        thread1.start();
        thread2.start();
    }
}

```
运行代码  
```
into waitA: Thread-0
exit waitA: Thread-0
wast time:4002
into waitA: Thread-1
exit waitA: Thread-1
wast time:8002
```
可以看出服务层从初始化到结束总共耗时8002ms，两个线程是互斥的。  
读读共享  
如果把lock.writeLock()缓存lock.readLock()  
输出结果为  
```
into waitA: Thread-0
into waitA: Thread-1
exit waitA: Thread-0
wast time:4003
exit waitA: Thread-1
wast time:4004
```
会发现总共耗时4004ms，两个线程是同步进行的  











northdbconfigini()
{






































 
类锁(synchronized 修饰静态的方法或代码块)  
　　由于一个class不论被实例化多少次，其中的静态方法和静态变量在内存中都只有一份。所以，一旦一个静态的方法被申明为synchronized。此类所有的实例化对象在调用此方法，共用同一把锁，我们称之为类锁。 　   
对象锁是用来控制实例方法之间的同步，类锁是用来控制静态方法（或静态变量互斥体）之间的同步。  
```
public class Test
{
　　 // 类锁：形式1
    public static synchronized void Method1()
    {
        System.out.println(＂我是类锁一号＂);
        try
        {
            Thread.sleep(500);
        } catch (InterruptedException e)
        {
            e.printStackTrace();
        }

    }
    // 类锁：形式2
    public void Method２()
    {
        synchronized (Test.class)
        {
            System.out.println(＂我是类锁二号＂);
            try
            {
                Thread.sleep(500);
            } catch (InterruptedException e)
            {
                e.printStackTrace();
            }

        }

    }
｝　
```
　对象锁（synchronized修饰方法或代码块）  
　　当一个对象中有synchronized method或synchronized block的时候调用此对象的同步方法或进入其同步区域时，就必须先获得对象锁。如果此对象的对象锁已被其他调用者占用，则需要等待此锁被释放。（方法锁也是对象锁） 　  　 　　　 
java的所有对象都含有1个互斥锁，这个锁由JVM自动获取和释放。线程进入synchronized方法的时候获取该对象的锁，当然如果已经有线程获取了这个对象的锁，那么当前线程会等待；synchronized方法正常返回或者抛异常而终止，JVM会自动释放对象锁。这里也体现了用synchronized来加锁的1个好处，方法抛异常的时候，锁仍然可以由JVM来自动释放。  
对象锁的两种形式：  
```
public class Test{
    // 对象锁：形式1(方法锁)
    public synchronized void Method1(){
        System.out.println("我是对象锁也是方法锁");
        try{
            Thread.sleep(500);
        } catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    // 对象锁：形式2（代码块形式）
    public void Method2(){
        synchronized (this){
            System.out.println("我是对象锁");
            try{
                Thread.sleep(500);
            } catch (InterruptedException e){
                e.printStackTrace();
            }
        }

    }
 ｝
```







　
