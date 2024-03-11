# mybatis
## 一级缓存和二级缓存
mybatis 现在是面试必问的，其中最主要的除了一些启动流程，基础语法，那么就是缓存问题了，在面试中也是常问的问题之一；

大家都知道mybatis是有二级缓存的，
其中一级缓存默认是开启的，二级缓存是要手动配置开启的，
但是本人这里不建议在实际生产中用mybatis的缓存，还是建议在外部实现自己的缓存，如使用redis等；
1：一级缓存是默认开启的；
2：底层其实是基于hashmap的本地内存缓存；
3：作用域是session（其实就相当于一个方法）；
4：当session关闭或者刷新的时候缓存清空；
5：不通sqlsession之间缓存互不影响；
1问题一：其实一级缓存也有数据一致性问题：
比如：我有一个更新操作对同一条数据，
如果是sqlsessionA进行了更新操作，则sqlsessionA对应的一级缓存被清空；
如果是sqlsessionB进行了更新操作，则此更新操作对改sqlsessionA不可见；
那么其实这个时候sqlsessionA再查的数据就是过期失效数据了；
就出现了数据不一致现象；

建议：
1：单个sqlsession的生命周期不能过长；
2：如果是对同一个语句更新尽量使用同一个sql，也就是同一个sqlsession；
3：建议关闭一级缓存，
怎么关闭呢？
在mybatis的全局配置文件中增加

<settiog name="localCacheScope" value="STATEMENT" />



