## CopyOnWriteArrayList概述

CopyOnWriteArrayList 是 Java 中的线程安全容器之一，其线程安全的原因主要是基于以下两个特点：

1. **不可变性**：
   - 在 CopyOnWriteArrayList 中，所有的修改操作（如添加、修改和删除）都不是直接在原始数据上进行的，而是在一个复制（或快照）上进行的。
   - 当需要进行修改时，先将原始数据复制一份，然后在副本上进行修改操作，修改完成后再将副本替换回原始数据。
   - 这种机制保证了在修改操作时，不会影响到正在进行迭代操作或其他线程正在访问的数据。
2. **写时复制**：
   - CopyOnWriteArrayList 的名称中就包含了 "CopyOnWrite" 这个词，意味着写操作时进行复制。
   - 当执行写操作时，会先将整个数据集复制一份，然后再进行修改，这样做的好处是在写操作期间，读操作可以并发执行而不受影响。
   - 复制操作可能会消耗一些额外的内存，但是由于写操作不会阻塞读操作，所以适用于读多写少的场景。

由于以上特点，CopyOnWriteArrayList 在多线程环境中具有很好的线程安全性：

- 读操作不需要加锁，可以并发执行，不会受到写操作的影响。
- 写操作虽然需要复制数据，但是由于写操作不频繁，所以对性能影响较小。
- 在需要遍历操作或读多写少的场景下，CopyOnWriteArrayList 是一种很好的选择，能够提供较好的线程安全性和性能表

### 类的继承关系

CopyOnWriteArrayList实现了List接口，List接口定义了对列表的基本操作；同时实现了RandomAccess接口，表示可以随机访问(数组具有随机访问的特性)；同时实现了Cloneable接口，表示可克隆；同时也实现了Serializable接口，表示可被序列化。

### 类的内部类

- COWIterator类

COWIterator表示迭代器，其也有一个Object类型的数组作为CopyOnWriteArrayList数组的快照，这种快照风格的迭代器方法在创建迭代器时使用了对当时数组状态的引用。此数组在迭代器的生存期内不会更改，因此不可能发生冲突，并且迭代器保证不会抛出 ConcurrentModificationException。创建迭代器以后，迭代器就不会反映列表的添加、移除或者更改。在迭代器上进行的元素更改操作(remove、set 和 add)不受支持。这些方法将抛出 UnsupportedOperationException。

### 类的属性

属性中有一个可重入锁，用来保证线程安全访问，还有一个Object类型的数组，用来存放具体的元素。当然，也使用到了反射机制和CAS来保证原子性的修改lock域。

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    // 版本序列号
    private static final long serialVersionUID = 8673264195747942595L;
    // 可重入锁
    final transient ReentrantLock lock = new ReentrantLock();
    // 对象数组，用于存放元素
    private transient volatile Object[] array;
    // 反射机制
    private static final sun.misc.Unsafe UNSAFE;
    // lock域的内存偏移量
    private static final long lockOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = CopyOnWriteArrayList.class;
            lockOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("lock"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

### 类的构造函数

- 默认构造函数

```java
public CopyOnWriteArrayList() {
    // 设置数组
    setArray(new Object[0]);
}
```

- `CopyOnWriteArrayList(Collection<? extends E>)`型构造函数　 该构造函数用于创建一个按 collection 的迭代器返回元素的顺序包含指定 collection 元素的列表。

```java
public CopyOnWriteArrayList(Collection<? extends E> c) {
    Object[] elements;
    if (c.getClass() == CopyOnWriteArrayList.class) // 类型相同
        // 获取c集合的数组
        elements = ((CopyOnWriteArrayList<?>)c).getArray();
    else { // 类型不相同
        // 将c集合转化为数组并赋值给elements
        elements = c.toArray();
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elements.getClass() != Object[].class) // elements类型不为Object[]类型
            // 将elements数组转化为Object[]类型的数组
            elements = Arrays.copyOf(elements, elements.length, Object[].class);
    }
    // 设置数组
    setArray(elements);
}
```

该构造函数的处理流程如下

- 判断传入的集合c的类型是否为CopyOnWriteArrayList类型，若是，则获取该集合类型的底层数组(Object[])，并且设置当前CopyOnWriteArrayList的数组(Object[]数组)，进入步骤③；否则，进入步骤②
- 将传入的集合转化为数组elements，判断elements的类型是否为Object[]类型(toArray方法可能不会返回Object类型的数组)，若不是，则将elements转化为Object类型的数组。进入步骤③
- 设置当前CopyOnWriteArrayList的Object[]为elements。

- `CopyOnWriteArrayList(E[])`型构造函数

该构造函数用于创建一个保存给定数组的副本的列表。

```java
public CopyOnWriteArrayList(E[] toCopyIn) {
    // 将toCopyIn转化为Object[]类型数组，然后设置当前数组
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
}
```

### CopyOnWriteArrayList的缺陷和使用场景

CopyOnWriteArrayList 有几个缺点：

- 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc
- 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个set操作后，读取到数据可能还是旧的,虽然CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求；

**CopyOnWriteArrayList 合适读多写少的场景，不过这类慎用**

因为谁也没法保证CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次add/set都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

### CopyOnWriteArrayList为什么并发安全且性能比Vector好?

Vector对单独的add，remove等方法都是在方法上加了synchronized; 并且如果一个线程A调用size时，另一个线程B 执行了remove，然后size的值就不是最新的，然后线程A调用remove就会越界(这时就需要再加一个Synchronized)。这样就导致有了双重锁，效率大大降低，何必呢。于是vector废弃了，要用就用CopyOnWriteArrayList 吧。

