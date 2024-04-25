# Java 数据结构
详细请转到[@pdai的博客](https://pdai.tech/md/java/thread/java-thread-x-juc-collection-BlockingQueue.html)
## 1. 基本数据结构
### 1.1 数组 (Array)
数组的优点:
* 存取速度快
数组的缺点:
* 事先必须知道数组的长度
* 插入删除元素很慢
* 空间通常是有限制的
* 需要大块连续的内存块
* 插入删除元素的效率很低

源码分析：
1、底层数据结构是Object
```
 transient Object[] elementData;
 private int size;
```
2、构造函数包括无参构造和有参数构造，有参构造时指定构造大小，或者直接复制已有的
```
public ArrayList(int initialCapacity)
public ArrayList()
public ArrayList(Collection<? extends E> c)

```
其中复制方式，public ArrayList(Collection<? extends E> c)，底层实现是通过Arrays.copyof接口来实现的
```
 elementData = Arrays.copyOf(elementData, size, Object[].class);
```
3、扩容机制
默认情况下：
ArrayList的容量为10，是由以下参数决定的
```
private static final int DEFAULT_CAPACITY = 10;
```
扩容机制触发条件：空间已满
扩容大小：1.5倍
源码调用路径
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/1a593e54b3584f0396ac94b4ad89a8a6.png)


其中，在add的时候，会调用 this.ensureCapacityInternal(this.size + 1);其中size+1是当前添加后的容量值，传入ensureExplicitCapacity方法，源码如下：
```
    private void ensureExplicitCapacity(int minCapacity) {
        ++this.modCount;
        //如果容量不足
        if (minCapacity - this.elementData.length > 0) {
            this.grow(minCapacity);
        }

```
这里可以看到容量不足时会调用 this.grow(minCapacity);
grow方式是最关键的方法
```
    private void grow(int minCapacity) {
        int oldCapacity = this.elementData.length;
        //扩容1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        //如果超过这个大容量要单独处理，return minCapacity > 2147483639 ? 2147483647 : 2147483639;
        if (newCapacity - 2147483639 > 0) {
            newCapacity = hugeCapacity(minCapacity);
        }
        //先把之前的copy过去
        this.elementData = Arrays.copyOf(this.elementData, newCapacity);
    }
```
扩完容之后就可以直接放入数据了
```
    public boolean add(E e) {
        this.ensureCapacityInternal(this.size + 1);
        this.elementData[this.size++] = e;
        return true;
    }
```

4、为什么remove性能不好
因为整体要平移，直接看源码
```
public boolean remove(Object o) {
        int index;
        if (o == null) {
            for(index = 0; index < this.size; ++index) {
                if (this.elementData[index] == null) {
                    this.fastRemove(index);
                    return true;
                }
            }
        } else {
            for(index = 0; index < this.size; ++index) {
                if (o.equals(this.elementData[index])) {
                //效率较高的数组拷贝方式,调用了System.arraycopy，实际上是一种o(n)的复杂度快速改变索引的方式
                    this.fastRemove(index);
                    return true;
                }
            }
        }

        return false;
    }
```
整体平移的时候，通过一个for循环挨个执行，时间复杂度是o(n)，加上this.fastRemove(index);是一种o(n)的拷贝方式，效率可想而知

5、其他API
indexOf(), lastIndexOf():获取元素的第一次出现的index。
get()：获取值
set()：设置值
addAll()：添加大部分数据
6、优化：
在使用ArrayList时如果新增数据，后面逐渐删除，会导致当前数组占用空间过大，无法清理，实际上里面只存储了几个数据的情况，因此，出现这种情况可以利用jdk提供的方法优化：
trimToSize()：底层数组的容量调整为当前列表保存的实际元素的大小的功能，源码如下：
```
 public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
     }
}
```
### 1.2 链表 (Linked List)
LinkedList同时实现了List接口和Deque接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列(Queue)，同时又可以看作一个栈(Stack)。这样看来，LinkedList简直就是个全能冠军。当你需要使用栈或者队列时，可以考虑使用LinkedList，一方面是因为Java官方已经声明不建议使用Stack类，更遗憾的是，Java里根本没有一个叫做Queue的类(它是个接口名字)。关于栈或队列，现在的首选是ArrayDeque，它有着比LinkedList(当作栈或队列使用时)有着更好的性能。
整体上，这是一个双向链表结构，在Linkedlist定义了三个数据
```
    transient int size;//当前数目
    transient LinkedList.Node<E> first;//指向链表的第一个
    transient LinkedList.Node<E> last;//指向链表的最后一个元素
```

这个链表的每个节点的数据结构
```
Node(LinkedList.Node<E> prev, E element, LinkedList.Node<E> next) {
this.item = element;//当前元素值
this.next = next;//后驱
this.prev = prev;//前驱
}
```
1、删除为什么这么快
removeFirst(), removeLast(), remove(e), remove(index)
这些方法用的是
```
public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```
核心方法为unlink，这个方法实际上是改变x这个节点的前驱指向x的后驱，同时将x的指向都变成了null，实际上是个o(1)复杂度的代码
```
 E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {// 第一个元素
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {// 最后一个元素
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null; // GC
        size--;
        modCount++;
        return element;
    }
```

2、add为什么不需要扩容
改变指向就好了，可以无限增长
add(E e)、add(int index, E element)
3、get()效率比较低
查找操作的本质是查找元素的下标，如源码所示，会从first遍历整个list，时间复杂度是o(n)；
```
 LinkedList.Node<E> node(int index) {
        LinkedList.Node x;
        int i;
        if (index < this.size >> 1) {
            x = this.first;

            for(i = 0; i < index; ++i) {
                x = x.next;
            }

            return x;
        } else {
            x = this.last;

            for(i = this.size - 1; i > index; --i) {
                x = x.prev;
            }

            return x;
        }
    }
```

4、其他API
getFirst(), getLast()
获取第一个元素， 和获取最后一个元素:
Queue 方法：
peek()：本质上获取第一个元素
poll()：本质上将数据从第一个元素删除，并获取第一个元素
remove()：调用的是removeFirst()
offer(E e)：调用的是add(e)
element()：调用的是getFirst()

Deque 方法:
offerFirst(E e)：调用addFirst(e)
offerLast(E e)：调用addLast(e)
peekFirst()： 获取fist节点的值
peekLast()：获取last节点的值
pollFirst()：删除第一个节点
pollLast()：删除最后一个节点
push(E e)：添加作为第一个节点
pop()：弹出第一个节点




### 1.3 栈 (Stack)
栈本质上是一个加了锁的数组（Vector），通过定义规则的方式，实现了先进后出逻辑，但是这个Vector方法由于性能问题已经算是个过时的接口，因此Stack也不再建议使用，这里不再多说，官方更建议将ArrayDeque来作为栈和队列来使用。


### 1.4 队列 (Queue)
#### 双向队列
Deque是"double ended queue", 表示双向的队列，英文读作"deck". Deque 继承自 Queue接口,除了支持Queue的方法之外，还支持insert, remove和examine操作，由于Deque是双向的，所以可以对队列的头和尾都进行操作，它同时也支持两组格式，一组是抛出异常的实现；另外一组是返回值的实现(没有则返回null)。
当把Deque当做FIFO的queue来使用时，元素是从deque的尾部添加，从头部进行删除的； 所以deque的部分方法是和queue是等同的。具体如下:
```
    //add调用的方法添加元素到尾部
    public void addLast(E e) {
        if (e == null) {
            throw new NullPointerException();
        } else {
            this.elements[this.tail] = e;
            if ((this.tail = this.tail + 1 & this.elements.length - 1) == this.head) {
                this.doubleCapacity();
            }

        }
    }
    //poll调用的方法，从头部获取元素
    public E pollFirst() {
        int h = this.head;
        E result = this.elements[h];
        if (result == null) {
            return null;
        } else {
            this.elements[h] = null;
            this.head = h + 1 & this.elements.length - 1;
            return result;
        }
    }
```
ArrayDeque和LinkedList是Deque的两个通用实现，由于官方更推荐使用AarryDeque用作栈和队列，只是底层的数据结构不一样，一个循环数组，一个是双向链表。
在这个循环数组中，为了满足可以同时在数组两端插入或删除元素的需求，数组的任何一点都可能被看作起点或者终点。
#### 优先级队列
这个优先级队列实际上是一个满足堆特性的数组结构。默认情况下是个小顶堆。
可以改为大顶堆
```
 PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```
在介绍PriorityQueue之前首先需要明确树结构与数组结构的关系：
```
leftNo = parentNo*2+1
rightNo = parentNo*2+2
parentNo = (nodeNo-1)/2
```
对于数组来说需要了解其初始与扩容机制
```
int newCapacity = oldCapacity + (oldCapacity < 64 ? oldCapacity + 2 : oldCapacity >> 1);
```
oldCapacity < 64 判断：如果当前容量小于 64，那么使用 oldCapacity + 2 作为新容量。这是为了在数组容量较小时，提供一些额外的空间，以避免频繁扩容。
oldCapacity >= 64 判断：如果当前容量大于等于 64，那么使用 oldCapacity >> 1（相当于 oldCapacity / 2）作为新容量。这是为了在容量较大时，以较小的步长逐渐扩容，降低扩容的速度。1.5倍

## 2. 树形数据结构

### 2.1 二叉树 (Binary Tree)
TreeMap(红黑树的Map结构)


### 2.2 堆 (Heap)
PriorityQueue，默认小顶堆
可以改为大顶堆
```
 PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```





## 3. 散列数据结构

### 3.1 哈希表 (Hash Map)
HashMap实现了Map接口，即允许放入key为null的元素，也允许插入value为null的元素；除该类未实现同步外，其余跟Hashtable大致相同；跟TreeMap不同，该容器不保证元素顺序，根据需要该容器可能会对元素重新哈希，元素的顺序也会被重新打散，因此不同时间迭代同一个HashMap的顺序可能会不同。 根据对冲突的处理方式不同，哈希表有两种实现方式，一种开放地址方式(Open addressing)，另一种是冲突链表方式(Separate chaining with linked lists)。

resize() 方法用于初始化数组或数组扩容，每次扩容后，容量为原来的 2 倍，并进行数据迁移。  
Java8 对 HashMap 进行了一些修改，最大的不同就是利用了红黑树，所以其由 数组+链表+红黑树 组成。  
根据 Java7 HashMap 的介绍，我们知道，查找的时候，根据 hash 值我们能够快速定位到数组的具体下标，但是之后的话，需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度，为 O(n)。为了降低这部分的开销，在 Java8 中，当链表中的元素达到了 8 个时，会将链表转换为红黑树，在这些位置进行查找的时候可以降低时间复杂度为 O(logN)。  

插入：  
put(K key, V value)方法是将指定的key, value对添加到map里。该方法首先会对map做一次查找，看是否包含该元组，如果已经包含则直接返回，查找过程类似于getEntry()方法；如果没有找到，则会通过addEntry(int hash, K key, V value, int bucketIndex)方法插入新的entry，插入方式为头插法。  
resize() 方法用于初始化数组或数组扩容，每次扩容后，容量为原来的 2 倍，并进行数据迁移。  
删除：  
remove(Object key)的作用是删除key值对应的entry，该方法的具体逻辑是在removeEntryForKey(Object key)里实现的。removeEntryForKey()方法会首先找到key值对应的entry，然后删除该entry(修改链表的相应引用)。查找过程跟getEntry()过程类似。  

获取：  
计算 key 的 hash 值，根据 hash 值找到对应数组下标: hash & (length-1)判断数组该位置处的元素是否刚好就是我们要找的，如果不是，走第三步判断该元素类型是否是 TreeNode，如果是，用红黑树的方法取数据，如果不是，走第四步遍历链表，直到找到相等(==或equals)的 key  

### 3.2 LinkedHashMap  

LinkedHashMap 是 Java 中的一个集合类，它是 HashMap 的子类。与普通的 HashMap 不同，LinkedHashMap 保留了元素插入的顺序。它使用一个双向链表来维护元素的顺序，该链表连接了所有的元素。因此，当你迭代 LinkedHashMap 时，元素的顺序与插入顺序一致。  
1、保留插入顺序： LinkedHashMap 会按照元素插入的顺序来维护元素的顺序，这是通过内部的双向链表实现的。  
2、性能类似于 HashMap： 在大多数操作上，LinkedHashMap 的性能与 HashMap 类似。它使用哈希表来实现快速的查找、插入和删除操作。  
3、迭代顺序： 迭代 LinkedHashMap 的时候，元素的顺序是按照插入的顺序。这与 HashMap 不同，后者的迭代顺序是不确定的。  
4、支持 LRU 缓存策略： LinkedHashMap 提供了一个构造函数，允许你创建一个有限容量的 LinkedHashMap，当达到容量上限时，会根据最近最少使用的原则删除最不经常使用的元素。  

### 3.3 TreeMap  
转载[TreeMap 源码解析](https://pdai.tech/md/java/collection/java-map-TreeMap&TreeSet.html)
TreeMap实现了SortedMap接口，也就是说会按照key的大小顺序对Map中的元素进行排序，key大小的评判可以通过其本身的自然顺序(natural ordering)，也可以通过构造时传入的比较器(Comparator)。
TreeMap底层通过红黑树(Red-Black tree)实现，也就意味着containsKey(), get(), put(), remove()都有着log(n)的时间复杂度。  

补充：  
红黑树：  
红黑树是一种近似平衡的二叉查找树，它能够确保任何一个节点的左右子树的高度差不会超过二者中较低那个的一倍。具体来说，红黑树是满足如下条件的二叉查找树(binary search tree):  
1、每个节点要么是红色，要么是黑色。  
3、根节点必须是黑色  
4、红色节点不能连续(也即是，红色节点的孩子和父亲都不能是红色)。  
5、对于每个节点，从该点至null(树尾端)的任何路径，都含有相同个数的黑色节点。  

左旋(Rotate Left)：  
左旋的过程是将x的右子树绕x逆时针旋转，使得x的右子树成为x的父亲，同时修改相关节点的引用。旋转之后，二叉查找树的属性仍然满足。  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/5685ccf12d4344fa922a7f5ba4c4a4ac.png)


左旋代码：  
```
private void rotateLeft(Entry<K,V> p) {  
    if (p != null) {
        Entry<K,V> r = p.right;
        p.right = r.left;
        if (r.left != null)
            r.left.parent = p;
        r.parent = p.parent;
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;
        r.left = p;
        p.parent = r;
    }
}
```
右旋：右旋的过程是将x的左子树绕x顺时针旋转，使得x的左子树成为x的父亲，同时修改相关节点的引用。旋转之后，二叉查找树的属性仍然满足。  
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/09440ebb0917460d9a157a8b214b9378.png)  

TreeMap中右旋代码如下:
```
private void rotateRight(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> l = p.left;
        p.left = l.right;
        if (l.right != null) l.right.parent = p;
        l.parent = p.parent;
        if (p.parent == null)
            root = l;
        else if (p.parent.right == p)
            p.parent.right = l;
        else p.parent.left = l;
        l.right = p;
        p.parent = l;
    }
}
```
get方法：  
get(Object key)方法根据指定的key值返回对应的value，该方法调用了getEntry(Object key)得到相应的entry，然后返回entry.value。因此getEntry()是算法的核心。算法思想是根据key的自然顺序(或者比较器顺序)对二叉查找树进行查找，直到找到满足k.compareTo(p.key) == 0的entry。  
```
final Entry<K,V> getEntry(Object key) {
    ......
    if (key == null)//不允许key值为null
        throw new NullPointerException();
    Comparable<? super K> k = (Comparable<? super K>) key;//使用元素的自然顺序
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)//向左找
            p = p.left;
        else if (cmp > 0)//向右找
            p = p.right;
        else
            return p;
    }
    return null;
}
```
put()方法：  
put(K key, V value)方法是将指定的key, value对添加到map里。该方法首先会对map做一次查找，看是否包含该元组，如果已经包含则直接返回，查找过程类似于getEntry()方法；如果没有找到则会在红黑树中插入新的entry，如果插入之后破坏了红黑树的约束条件，还需要进行调整(旋转，改变某些节点的颜色)。  
```
public V put(K key, V value) {
	......
    int cmp;
    Entry<K,V> parent;
    if (key == null)
        throw new NullPointerException();
    Comparable<? super K> k = (Comparable<? super K>) key;//使用元素的自然顺序
    do {
        parent = t;
        cmp = k.compareTo(t.key);
        if (cmp < 0) t = t.left;//向左找
        else if (cmp > 0) t = t.right;//向右找
        else return t.setValue(value);
    } while (t != null);
    Entry<K,V> e = new Entry<>(key, value, parent);//创建并插入新的entry
    if (cmp < 0) parent.left = e;
    else parent.right = e;
    fixAfterInsertion(e);//调整
    size++;
    return null;
}
```
上述代码的插入部分并不难理解: 首先在红黑树上找到合适的位置，然后创建新的entry并插入(当然，新插入的节点一定是树的叶子)。难点是调整函数fixAfterInsertion()，前面已经说过，调整往往需要1.改变某些节点的颜色，2.对某些节点进行旋转。  

###  ConcurrentHashMap详解  
JDK1.7之前的ConcurrentHashMap使用分段锁机制实现，JDK1.8则使用数组+链表+红黑树数据结构和CAS原子操作实现ConcurrentHashMap；本文将分别介绍这两种方式的实现方案及其区别。  
#### ConcurrentHashMap - JDK 1.7  
ConcurrentHashMap在对象中保存了一个Segment数组，即将整个Hash表划分为多个分段；而每个Segment元素，即每个分段则类似于一个Hashtable；这样，在执行put操作时首先根据hash算法定位到元素属于哪个Segment，然后对该Segment加锁即可。因此，ConcurrentHashMap在多线程并发编程中可是实现多线程put操作。    

Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。  
ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写。  

初始化：  
loadFactor: 负载因子，是 0.75，得出初始阈值为 1.5，也就是以后插入第一个元素不会触发扩容，插入第二个会进行第一次扩容  
initialCapacity: 初始容量  

并发问题安全性：  
put 操作的线程安全性。   
使用了 CAS 来初始化 Segment 中的数组。添加节点到链表的操作是插入到表头的，所以，如果这个时候 get 操作在链表遍历的过程已经到了中间，是不会影响的。  
当然，另一个并发问题就是 get 操作在 put 之后，需要保证刚刚插入表头的节点被读取，这个依赖于 setEntryAt 方法中使用的 UNSAFE.putOrderedObject。  
扩容。扩容是新创建了数组，然后进行迁移数据，最后面将 newTable 设置给属性 table。所以，如果 get 操作此时也在进行，那么也没关系，如果 get 先行，那么就是在旧的 table 上做查询操作；而 put 先行，那么 put 操作的可见性保证就是 table 使用了 volatile 关键字。  
remove 操作的线程安全性。   
get 操作需要遍历链表，但是 remove 操作会"破坏"链表。如果 remove 破坏的节点 get 操作已经过去了，那么这里不存在任何问题。如果 remove 先破坏了一个节点，分两种情况考虑。  
 1、如果此节点是头节点，那么需要将头节点的 next 设置为数组该位置的元素，table 虽然使用了 volatile 修饰，但是 volatile 并不能提供数组内部操作的可见性保证，所以源码中使用了 UNSAFE 来操作数组，请看方法 setEntryAt。  
 2、如果要删除的节点不是头节点，它会将要删除节点的后继节点接到前驱节点中，这里的并发保证就是 next 属性是 volatile 的。  

#### ConcurrentHashMap - JDK 1.8

在JDK1.8中，ConcurrentHashMap选择了与HashMap类似的数组+链表+红黑树的方式实现，而加锁则采用CAS和synchronized实现。  

## 4. 并发数据结构（JUC集合）

### 4.1 ConcurrentHashMap

### 4.2 CopyOnWriteArrayList
CopyOnWriteArrayList实现了List接口，List接口定义了对列表的基本操作；同时实现了RandomAccess接口，表示可以随机访问(数组具有随机访问的特性)；同时实现了Cloneable接口，表示可克隆；同时也实现了Serializable接口，表示可被序列化。  

CopyOnWriteArrayList 是 Java 中并发编程提供的线程安全的集合类之一，它的特点是在进行写操作时会创建一个新的复制（copy）来保证线程安全。以下是 CopyOnWriteArrayList 的主要原理和特点：  
1、写时复制： 当有写操作（如添加、删除元素）发生时，CopyOnWriteArrayList 会创建一个新的数组（复制原数组），在新数组上执行写操作，然后将新数组替换原来的数组。这样可以确保写操作不会影响正在进行的读操作，因为读操作仍然在原数组上进行。  
2、读操作不需要加锁： 由于读操作不涉及修改集合内容，而是在不变的原数组上进行，因此读操作不需要加锁。这使得在读多写少的场景中，CopyOnWriteArrayList 的性能相对较好。  
3、适用于读多写少的场景： CopyOnWriteArrayList 的设计适用于那些读操作频繁，而写操作相对较少的场景。这是因为在写时需要进行数组复制，可能会引起一些性能开销。  
4、弱一致性： CopyOnWriteArrayList 提供的是弱一致性，即在写操作完成后，对于读操作来说，并不一定能立即看到最新的修改。这是因为读操作仍然可能在旧数组上进行，直到写操作完成并新数组替换了旧数组。  
5、不支持并发修改的迭代器： 由于写操作会创建新的数组，旧数组上的迭代器可能会抛出 ConcurrentModificationException 异常。因此，CopyOnWriteArrayList 的迭代器是不支持并发修改的。  


CopyOnWriteArrayList的缺陷和使用场景CopyOnWriteArrayList 有几个缺点：由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个set操作后，读取到数据可能还是旧的,虽然CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求；  
CopyOnWriteArrayList 合适读多写少的场景，不过这类慎用因为谁也没法保证CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次add/set都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。  

### 4.3 ConcurrentLinkedQueue
ConcurerntLinkedQueue一个基于链接节点的无界线程安全队列。此队列按照 FIFO(先进先出)原则对元素进行排序。队列的头部是队列中时间最长的元素。队列的尾部 是队列中时间最短的元素。新的元素插入到队列的尾部，队列获取操作从队列头部获得元素。当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue是一个恰当的选择。此队列不允许使用null元素。  

ConcurrentLinkedQueue 是 Java 中并发编程提供的线程安全的队列实现。它基于非阻塞算法，使用 CAS（Compare and Swap）等原子操作来确保线程安全。以下是 ConcurrentLinkedQueue 的主要特点和原理：  
非阻塞算法： ConcurrentLinkedQueue 使用非阻塞算法，避免了使用锁的情况，因此具有较好的并发性能。它通过 CAS 操作等方式来保证多个线程可以同时进行读和写操作。  

1、无界队列： ConcurrentLinkedQueue 是无界队列，它没有固定的容量限制。可以不受限制地添加元素。  
2、基于链表： 内部使用单向链表来组织队列元素。新的元素总是被添加到队列的尾部，而移除元素则发生在队列的头部。  
3、线程安全的入队和出队操作： ConcurrentLinkedQueue 提供了线程安全的入队（offer和add）和出队（poll和remove）操作。这意味着多个线程可以同时进行这些操作而不需要额外的同步。  
4、迭代器支持弱一致性： ConcurrentLinkedQueue 的迭代器是弱一致性的，它不会抛出 ConcurrentModificationException 异常。因此，迭代器可能看到队列的部分修改，而不一定是最新状态。  
5、适用于生产者-消费者模型： ConcurrentLinkedQueue 在生产者-消费者模型中是一个常用的选择，特别是当多个线程并发地生产和消费元素时。  

### 4.4 BlockingQueue
BlockingQueue大家族有哪些? ArrayBlockingQueue, DelayQueue, LinkedBlockingQueue, SynchronousQueue...  
BlockingQueue 通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。  
一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。
负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。  
#### ArrayBlockingQueue:
基于数组的阻塞队列实现。  
有界队列，必须指定容量。  
使用独占锁实现线程安全。  
```
BlockingQueue<Integer> arrayBlockingQueue = new ArrayBlockingQueue<>(10);
```
#### LinkedBlockingQueue:
基于链表的阻塞队列实现。  
可以选择有界或无界。  
如果不指定容量，默认是无界队列。  
使用两个锁（读锁和写锁）实现线程安全。  
```
BlockingQueue<String> linkedBlockingQueue = new LinkedBlockingQueue<>();
```
#### PriorityBlockingQueue:
一个支持优先级的无界阻塞队列。    
元素按照它们的自然顺序或者根据构造函数中提供的比较器来进行排序。   
不保证同优先级元素的顺序。  
```
BlockingQueue<Task> priorityBlockingQueue = new PriorityBlockingQueue<>();
```
#### DelayQueue:
用于存放实现了 Delayed 接口的元素，这些元素只能在其指定的延迟时间过后才能被消费。  
通常用于定时任务调度。  
```
BlockingQueue<DelayedTask> delayQueue = new DelayQueue<>();
```
#### SynchronousQueue:
一个不存储元素的阻塞队列。  
生产者线程必须等待消费者线程取走元素，反之亦然。  
用于直接传递任务或数据，通常在生产者和消费者线程之间进行交互。  
```
BlockingQueue<String> synchronousQueue = new SynchronousQueue<>(); 
```

