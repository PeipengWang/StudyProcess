所需jar包：
self4j-api-1.7.25.jar
ehcache-2.10.6.jar

或者直接利用Maven导入[地址](https://mvnrepository.com/artifact/net.sf.ehcache/ehcache)

```xml
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.6</version>
</dependency>

```
配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>

<ehcache>

    <diskStore path="java.io.tmpdir/Tmp_EhCache"/>

    <defaultCache
            maxEntriesLocalHeap="1000"
            eternal="false"
            timeToIdleSeconds="1200"
            timeToLiveSeconds="3000"
            diskSpoolBufferSizeMB="30"
            maxEntriesLocalDisk="10000"
            diskExpiryThreadIntervalSeconds="1200"
            memoryStoreEvictionPolicy="LRU"
            statistics="false">
        <persistence strategy="localTempSwap"/>
    </defaultCache>

    <cache name="a"
           eternal="false"
           timeToIdleSeconds="1200"
           timeToLiveSeconds="3000"
           maxElementsInMemory="1000"
           memoryStoreEvictionPolicy="FIFO"/>

</ehcache>
<!--
   缓存配置
      diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
             user.home – 用户主目录
             user.dir  – 用户当前工作目录
             java.io.tmpdir – 默认临时文件路径
     defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
        必须：
        name:缓存名称。
        maxElementsInMemory：缓存最大个数，0表示没有限制
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。
        maxElementsOnDisk：在DiskStore中的最大对象数量，如为0，则没有限制
        overflowToDisk：是否当memory中的数量达到限制后，保存到Disk
        可选：
        timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也       就是可闲置时间无穷大。
        timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
        diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
        memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
        clearOnFlush：内存数量最大时是否清除。
           -->
```

```java
import net.sf.ehcache.Cache;
import net.sf.ehcache.CacheManager;
import net.sf.ehcache.Element;

public class testCache {
    public static void main(String[] args) {
        String path = System.getProperty("user.dir");
        //根据xml文件创建Cache缓存
        CacheManager  cacheManager =  CacheManager.create(path+"/src/Ehcache.xml");
        Cache c = cacheManager.getCache("a");//获取指定cache
        Element e = new Element("key1", "value1");
        c.put(e);
        System.out.println("读取缓存数据");
        Element e2 = c.get("key1");
        System.out.println(e2);
        System.out.println("清除缓存数据中。。。");
        c.flush();
        System.out.println("再次读取缓存数据");
        Element e3 = c.get("key1");
        System.out.println(e3);
        System.out.println("关闭缓存管理器");
        cacheManager.shutdown();
    }
}

```
测试结果

```html
读取缓存数据
[ key = key1, value=value1, version=1, hitCount=1, CreationTime = 1632645970559, LastAccessTime = 1632645970560 ]
清除缓存数据中。。。
再次读取缓存数据
null
关闭缓存管理器

```
遇到的坑：
jar包导入需要导入self-api，不然在执行create方法时会失效
