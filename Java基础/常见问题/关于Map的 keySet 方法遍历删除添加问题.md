对于读取Map的所有键值集合有一个合适的方法：Set<Integer> sets  = map.keySet();
如果想要直接遍历可以采用：

```java
        Set<Integer> sets  = map.keySet();
        Iterator<Integer> it=sets.iterator();
        while(it.hasNext()){
            Integer entry=it.next();
            System.out.println("key="+entry+","+"value="+map.get(entry));
        }
```
这里通过 Set 集合方式可以获取所有的 map 的 key 值，通过 key 可以依次对数据进行获取，**但是，不能增删**
实例：

```java
        Map<Integer,String> map = new HashMap<Integer,String>();
        map.put(1, "aaa");
        map.put(2, "bbb");
        map.put(3, "ccc");
        map.put(4, "ddd");
        Set<Integer> sets  = map.keySet();
        Iterator<Integer> it=sets.iterator();
        while(it.hasNext()){
            Integer entry=it.next();
            System.out.println("key="+entry+","+"value="+map.get(entry));
        }
```
这时候获取数据：
key=1,value=aaa
key=2,value=bbb
key=3,value=ccc
key=4,value=ddd

但是，当删除操作时，错误实例

```java
        Map<Integer,String> map = new HashMap<Integer,String>();
        map.put(1, "aaa");
        map.put(2, "bbb");
        map.put(3, "ccc");
        map.put(4, "ddd");
        Set<Integer> sets  = map.keySet();
        List<Integer> lists = new LinkedList<>();
        Iterator<Integer> it=sets.iterator();
        while(it.hasNext()){
            Integer entry=it.next();
            if(entry == 3)
            {
                map.remove(entry);
            }
            System.out.println("key="+entry+","+"value="+map.get(entry));
        }
```
出现异常：
key=1,value=aaa
key=2,value=bbb
key=3,value=null
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)
	at java.util.HashMap$KeyIterator.next(HashMap.java:1469)
	at testKeySet.main(testKeySet.java:14)

当删除 3 的map后，再次读取 set 会报错，原因分析：
debug程序发现，sets 与 map所指向的地址是一样的，当删除 map 值后相应的 sets 值也会丢失，这时候再读取下一个就会出现问题了。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/51782860534b4466b05ef9de4ab30df9.png)
想要解决这个问题，可以考虑在读取到 sets 之后重新定义一个 lists 集合，把 sets 里的数据放进去来遍历增删。
代码如下:

```java
 Map<Integer,String> map = new HashMap<Integer,String>();
        map.put(1, "aaa");
        map.put(2, "bbb");
        map.put(3, "ccc");
        map.put(4, "ddd");
        Set<Integer> sets  = map.keySet();
        List<Integer> lists = new LinkedList<>();
        for(Integer set:sets){
            lists.add(set);
        }
        for(Integer list:lists){
            if(list == 3) {
                map.remove(list);
            }
        }
        Iterator<Integer> it=sets.iterator();
        while(it.hasNext()){
            Integer entry=it.next();
            System.out.println("key="+entry+","+"value="+map.get(entry));
        }
```
此时可以正常显示：
key=1,value=aaa
key=2,value=bbb
key=4,value=ddd


后记：
遍历 Map 的方法

```java
for(Map map:maps){
}
```

迭代器

```java
        Iterator<Map.Entry<Integer,String>> it=map.entrySet().iterator();
        while(it.hasNext()){
            Map.Entry<Integer,String> entry=it.next();
            System.out.println("key="+entry.getKey()+","+"value="+entry.getValue());
        }
```
