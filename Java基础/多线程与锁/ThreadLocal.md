# ThreadLocal
ThreadLocal表示线程的“局部变量”，它确保每个线程的ThreadLocal变量都是各自独立的，它提供了一种方法来创建只能被当前线程访问的变量。这意味着如果有两个不同的线程访问同一个ThreadLocal变量，那么这两个线程将不能看到彼此的值，这就是“线程本地”的含义。

ThreadLocal适合在一个线程的处理流程中保持上下文（避免了同一参数在所有方法中传递）；
使用ThreadLocal要用try ... finally结构，并在finally中清除。

首先看一个线程不安全的实例
```
    private static SimpleDateFormat f = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static void main(String[] args) {
        while (true) {
            new Thread(() -> {
                String dateStr = f.format(new Date());
                try {
                    Date parseDate = f.parse(dateStr);
                    String dateStrCheck = f.format(parseDate);
                    boolean equals = dateStr.equals(dateStrCheck);
                    if (!equals) {
                        System.out.println(equals + " " + dateStr + " " + dateStrCheck);
                    } else {
                        System.out.println(equals);
                    }
                } catch (ParseException e) {
                    System.out.println(e.getMessage());
                }
            }).start();
        }
    }
```
在代码中每个线程都会 对f进行操作，会导致f这个对象发生变化，由于f这个对象是共享变量，各个线程之间互相影响，由此产生了三种情况
输出结果如下，正常情况为true，如果两个线程分别修改了数据，那么可能是false，当然，由于还处在中间过程，还会导致异常情况，那么该如何解决这个问题呢，就需要ThreadLocal了。
```
true
true
false 2024-01-31 10:09:52 0000-01-31 10:09:52
false 2024-01-31 10:09:52 0052-01-31 10:09:52
true
true
Exception in thread "Thread-1766" java.lang.NumberFormatException: For input string: ""
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Long.parseLong(Long.java:601)
	at java.lang.Long.parseLong(Long.java:631)
	at java.text.DigitList.getLong(DigitList.java:195)
	at java.text.DecimalFormat.parse(DecimalFormat.java:2084)
	at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1869)
	at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
	at java.text.DateFormat.parse(DateFormat.java:364)
	at shujujiegou.testThreadLocal.lambda$main$0(testThreadLocal.java:16)
	at java.lang.Thread.run(Thread.java:748)
```
现在已经知道ThreadLocal是本地变量的意思，相当于把共享变量与本地进行了隔离，每个线程复制一份，因此在操作过程中互不干扰  
写法如下：
```
private static ThreadLocal<SimpleDateFormat> threadLocal = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
public static void main(String[] args) {
    while (true) {
        new Thread(() -> {
            String dateStr = threadLocal.get().format(new Date());
            try {
                Date parseDate = threadLocal.get().parse(dateStr);
                String dateStrCheck = threadLocal.get().format(parseDate);
                boolean equals = dateStr.equals(dateStrCheck);
                if (!equals) {
                    System.out.println(equals + " " + dateStr + " " + dateStrCheck);
                } else {
                    System.out.println(equals);
                }
            } catch (ParseException e) {
                System.out.println(e.getMessage());
            }
        }).start();
    }
}
 
```
ThreadLocal数据结构  
1、它是一个数组结构。  
3、Entry，这里没用再打开，其实它是一个弱引用实现，static class Entry extends WeakReference<ThreadLocal<?>>。这说明只要没用强引用存在，发生GC时就会被垃圾回收。  
3、数据元素采用哈希散列方式进行存储，不过这里的散列使用的是 斐波那契（Fibonacci）散列法，后面会具体分析。  
4、另外由于这里不同于HashMap的数据结构，发生哈希碰撞不会存成链表或红黑树，而是使用开放寻址进行存储。也就是同一个下标位置发生冲突时，则+1向后寻址，直到找到空位置或垃圾回收位置进行存储。  
6、大于2/3就会扩容  
7、探测式清理，其实这也是非常耗时。为此我们在使用 ThreadLocal 一定要记得 new ThreadLocal<>().remove(); 操作。避免弱引用发生GC后，导致内存泄漏的问题。  
底层数据结构  
它的底层数据结构通常是一个 Map，其中线程是键，线程局部变量是值。Java 中的 ThreadLocal 类通过使用线程作为键，可以为每个线程存储一个独立的变量副本，从而实现线程隔离。  
在 ThreadLocal 的内部实现中，一般会使用一个 ThreadLocalMap 类型的实例来存储线程局部变量。ThreadLocalMap 是一个自定义的哈希表，它的键是 ThreadLocal 实例，值是线程局部变量的值。由于每个线程都有自己的 ThreadLocalMap 实例，所以每个线程都可以独立存取其对应的线程局部变量。  
总的来说，ThreadLocal 底层的数据结构就是一个 Map，它将线程作为键与线程局部变量的值进行关联，从而实现了线程隔离的效果。  
底层结构看这篇  
https://blog.csdn.net/qq_38599840/article/details/113850849  
