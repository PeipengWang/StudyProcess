# java的atomic原子引用

在Java中，java.util.concurrent.atomic 包提供了一些原子类，主要用于在不需要显式加锁的情况下执行原子操作。以下是一些常用的原子类：
1、AtomicInteger（原子整数）：

提供对整数的原子操作。
示例：

```
AtomicInteger atomicInt = new AtomicInteger(0);
int incrementedValue = atomicInt.incrementAndGet();
```

2、AtomicLong（原子长整数）：
提供对长整数的原子操作。
示例：

```
AtomicLong atomicLong = new AtomicLong(0);
long incrementedValue = atomicLong.incrementAndGet();
```

3、AtomicBoolean（原子布尔值）：
提供对布尔值的原子操作。
示例：

```
AtomicBoolean atomicBoolean = new AtomicBoolean(true);
boolean previousValue = atomicBoolean.getAndSet(false);
```

4、AtomicReference（原子引用）：

提供对通用对象引用的原子操作。
示例：

```
AtomicReference<String> atomicReference = new AtomicReference<>("初始值");
String updatedValue = atomicReference.getAndSet("新值");
AtomicStampedReference（带时间戳的原子引用）：
```

5、在 AtomicReference 的基础上添加了一个时间戳，用于解决ABA问题。

示例：

```
AtomicStampedReference<String> atomicStampedReference = new AtomicStampedReference<>("初始值", 0);
boolean success = atomicStampedReference.compareAndSet("初始值", "新值", 0, 1);
```

这些原子类提供了原子性的复合操作，可用于并发编程，避免了显式使用锁时可能出现的竞态条件。它们在需要在没有显式同步的情况下执行原子操作的场景中特别有用。
