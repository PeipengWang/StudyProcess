# redis
Remote Dictionary Server(Redis) 是一个开源的由Salvatore Sanfilippo使用ANSI C语言开发的key-value数据存储服务器。其值（value）可以是 字符串(String), 哈希(Map), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型，所以它通常也被称为数据结构服务器。是一款高性能的NOSQL系列的非关系型数据库。  
## Redis特点
redis足够简单和稳定  
支持丰富的数据结构  
内存存储读写性能优秀  
提供持久化的支持  
支持事务操作  
提供主从复制功能  
Redis的典型应用场景：  
一：缓存热点数据  
热点数据（经常会被查询，但是不经常被修改或者删除的数据），首选是使用redis缓存，redis的性能非常优秀。  
二：计数器  
诸如统计点击数、访问数、点赞数、评论数、浏览数等应用，由于单线程，可以避免并发问题，保证数据的正确性，并且100%毫秒级性能，同时开启Redis持久化，以便于持久化数据。  
三：单线程机制  
验证前端的重复请求，可以自由扩展类似情况），可以通过redis进行过滤，比如，每次请求将Request IP、参数、接口等hash作为key存储redis（幂等性请求），设置多长时间有效期，然后下次请求过来的时候先在redis中检索有没有这个key，进而验证是不是一定时间内过来的重复提交；再比如，限制用户登录的次数，比如一天错误登录次数10次等。  
秒杀系统，基于redis是单线程特征，防止出现数据库超卖；  
全局增量ID生成等；  
四：排行  
谁得分高谁排名在前，比如点击率最高、活跃度最高、销售数量最高、投票最高的前10名排行等等；  
五：分布式锁  
使用redis可以实现分布式锁，为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：  
互斥性，在任意时刻，只有一个客户端能持有锁。  
不会发生死锁，即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。  
具有容错性，只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。  
解铃还须系铃人，加锁和解锁必须是同一个客户端，客户端不能解他人加的锁。  
六：Session存储  
使用Redis的进行会话缓存（session cache）是非常常见的一种场景。用Redis缓存会话比其他存储（如Memcached）的优势在于：Redis提供持久化，目前大量的方案均采用了redis作为session的存储方案。  
## 基本数据操作    
### 字符串  
set ­­ 设置key对应的值为string类型的value。  
> set name itcast  
setnx ­­ 将key设置值为value，如果key不存在，这种情况下等同SET命令。 当key存在时，什么也不做。SETNX是”SET if Not eXists”的简写。  
> get name  
"itcast"  
> setnx name itcast_new    
(integer)0  
>get name  
"itcast"  
setex ­­ 设置key对应字符串value，并且设置key在给定的seconds时间之后超时过期。  
> setex color 10 red  
> get color  
"red"  
10秒后...  
> get color (nil)  
setrange ­­ 覆盖key对应的string的一部分，从指定的offset处开始，覆盖value的长度。  
127.0.0.1:6379> set key1 value1  
OK  
127.0.0.1:6379> setrange key1 1 value1  
(integer) 7  
127.0.0.1:6379> get value1  
"vvalue1"  
127.0.0.1:6379>STRLEN email  
(integer) 7  
mset ­­ 一次设置多个key的值,成功返回ok表示所有的值都设置了,失败返回0表示没有任何值被设置。  
127.0.0.1:6379> mset key1 value1 key2 value3  
OK  
127.0.0.1:6379> get key1  
"value1"  
127.0.0.1:6379> get key2  
"value3"  
mget ­­ 一次获取多个key的值,如果对应key不存在,则对应返回nil。   
127.0.0.1:6379> mget key1 key2 key3   
1) "value1"  
2) "value3"  
3) (nil)  
msetnx ­­ 对应给定的keys到他们相应的values上。只要有一个key已经存在，MSETNX一个操作都不会执行。  
getset ­­ 设置key的值,并返回key的旧值  
> get name  
"itcast"  
> getset name itcast_new  
"itcast"  
> get name  
"itcast_new"  
GETRANGE key start end ­­ 获取指定key的value值的子字符串。是由start和end位移决定的  
> getrange name 0 4  
  "itcas"  
  incr ­­ 对key的值加1操作  

> set age 20   
> incr age   
(integer) 21  
incrby ­­ 同incr类似,加指定值 ,key不存在时候会设置key,并认为原来的value是 0  
> incrby age 5  
  (integer) 26  
> incrby age1111 5  
(integer) 5  
> get age1111  
"5"  
decr ­­ 对key的值做的是减减操作,decr一个不存在key,则设置key为­1  
decrby ­­ 同decr,减指定值  
append ­­ 给指定key的字符串值追加value,返回新字符串值的长度。例如我们向name的值追加一个"redis"字符串:  
127.0.0.1:6379> get name  
"itcast_new"  
127.0.0.1:6379> append name "value"  
(integer) 15  
127.0.0.1:6379> get name  
"itcast_newvalue"  
127.0.0.1:6379>  
### HASH  
HASH 哈希  
Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。  
Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。  
HSET key field value ­­ 设置 key 指定的哈希集中指定字段的值  
hget ­­ 获取指定的hash field。  
HSETNX key field value ­­ 只在 key 指定的哈希集中不存在指定的字段时，设置字段的值。如果 key 指定的哈希集不存在，会创建一个新的哈希集并与 key 关联。如果字段已存在，该操作无效果。  
HSETNX key field value ­­ 只在 key 指定的哈希集中不存在指定的字段时，设置字段的值。如果 key 指定的哈希集不存在，会创建一个新的哈希集并与 key 关联。如果字段已存在，该操作无效果。  
hmset ­­ 同时设置hash的多个field。     
hmget ­­ 获取全部指定的hash filed。  
hincrby ­­ 指定的hash filed 加上给定值。  
hexists ­­ 测试指定field是否存在。  
hdel 从 key 指定的哈希集中移除指定的域  
hlen ­­ 返回指定hash的field数量。  
hkeys ­­ 返回hash的所有field。  
hvals ­­ 返回hash的所有value。  
hgetall ­­ 获取某个hash中全部的filed及value。  
HSTRLEN -- 返回 hash指定field的value的字符串长度  
### List 列表  
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）  
一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。  
RPUSH key value [value ...]  
向存于 key 的列表的尾部插入所有指定的值。如果 key 不存在，那么会创建一个空的列表然后再进行 push 操作。  
LPOP key  
移除并且返回 key 对应的 list 的第一个元素。  
LTRIM key start stop  
修剪(trim)一个已存在的 list，这样 list 就会只包含指定范围的指定元素。  
start 和 stop 都是由0开始计数的， 这里的 0 是列表里的第一个元素（表头），1 是第二个元素，以此类推。  
例如： LTRIM foobar 0 2 将会对存储在 foobar 的列表进行修剪，只保留列表里的前3个元素。  
start 和 end 也可以用负数来表示与表尾的偏移量，比如 -1 表示列表里的最后一个元素， -2 表示倒数第二个，等等。  
应用场景:  
1.取最新N个数据的操作  
比如典型的取你网站的最新文章，通过下面方式，我们可以将最新的5000条评论的ID放在Redis的List集合中，并将超出集合部分从数据库获取  
### Set 集合    
Set 就是一个集合,集合的概念就是一堆不重复值的组合。利用 Redis 提供的 Set 数据结构,可以存储一些集合性的数据。  
比如在 微博应用中,可以将一个用户所有的关注人存在一个集合中,将其所有粉丝存在一个集合。  
因为 Redis 非常人性化的为集合提供了 求交集、并集、差集等操作, 那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能, 对上面的所有集合操作,你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。
SADD key member [member ...]  
添加一个或多个指定的member元素到集合的 key中  
SCARD key  
返回集合存储的key的基数 (集合元素的数量).    
SDIFF key [key ...]  
返回一个集合与给定集合的差集的元素.

### Sorted Set 有序集合  
Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。  
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。  
有序集合的成员是唯一的,但分数(score)却可以重复。  
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。  
ZADD key score member  
将所有指定成员添加到键为key有序集合（sorted set）里面  
ZCOUNT key min max  
返回有序集key中，score值在min和max之间(默认包括score值等于min或max)的成员  
ZINCRBY key increment member  
为有序集key的成员member的score值加上增量increment  
应用场景  
1.带有权重的元素,LOL游戏大区最强王者  
2 排行榜  
### 事务  
Redis事务允许一组命令在单一步骤中执行。事务有两个属性，说明如下：  
事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。  
Redis事务是原子的。原子意味着要么所有的命令都执行，要么都不执行；  
一个事务从开始到执行会经历以下三个阶段：  
开始事务  
命令入队  
执行事务  
redis 127.0.0.1:6379> MULTI  
OK  
List of commands here  
redis 127.0.0.1:6379> EXEC  
### Redis 数据备份与恢复  
Redis SAVE 命令用于创建当前数据库的备份。  
语法  
redis Save 命令基本语法如下：  
redis 127.0.0.1:6379> SAVE  
实例  
redis 127.0.0.1:6379> SAVE   
OK  
该命令将在 redis 安装目录中创建dump.rdb文件。  
恢复数据  
如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：  
redis 127.0.0.1:6379> CONFIG GET dir  
1) "dir"  
2) "/home/python"  
以上命令 CONFIG GET dir 输出的 redis 安装目录为 /home/python。  
Bgsave  
创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。  
实例  
127.0.0.1:6379> BGSAVE  
Background saving started  
  










