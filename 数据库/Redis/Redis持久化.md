# Redis持久化
## RDB
### RDB是什么？
在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是键快照文件直接读到内存里。
### 备份是如何执行的？
Redis会单独创建（fork）一个子进程进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束后，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就是确保了极高的性能，如果需要进行大规模的恢复，且对数据恢复的完整性不是非常敏感，那RDB方式要不AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。
### 配置
在redis.conf中，可以修改rdb备份文件的名称，默认为dump.rdb  
在redis.conf中，rdb文件的保存的目录是可以修改的，默认为Redis启动命令所在的目录  
save用来配置备份的规则  
save的格式： save 秒钟 写操作次数  
默认是1分钟内修改了1万次，或5分钟内需修改了10次，或30分钟内修改了1次。  
示例：设置20秒内有最少有3次key发生变化，则进行备份   save 20 3  
方式2：手动执行命令备份（save | bgsave）  
有2个命令可以触发备份。  
save：save时只管保存，其他不管，全部阻塞，手动保存，不建议使用。  
bgsave：redis会在后台异步进行快照操作，快照同时还可以响应客户端情况。  
可以通过 lastsave 命令获取最后一次成功生成快照的时间。  
方式3：flushall命令  
执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义。  
### rdb的备份和恢复
1、 先通过config get dir 查询rdb文件的目录  
2、 然后将rdb的备份文件 *.rdb 文件拷贝到别的地方  
3、 rdb的恢复  
关闭redis  
先把备份的文件拷贝到工作目录 cp dump2.rdb dump.rdb  
启动redis，备份数据直接加载，数据被恢复  
### 优势
适合大规模数据恢复  
对数据完整性和一致性要求不高更适合使用  
节省磁盘空间  
恢复速度快  
### 劣势
Fork的时候，内存中的数据会被克隆一份，大致2倍的膨胀，需要考虑  
虽然Redis在fork的时候使用了写时拷贝技术，但是如果数据庞大时还是比较消耗性能  
在备份周期在一定间隔时间做一次备份，所以如果Redis意外down的话，就会丢失最后一次快照后  
所有修改  
## redis持久化之AOF  
以日志的形式来记录每个写操作（增量保存），将redis执行过的所有写指令记录下来（读操作不记录），只允追加文件但不可改写文件，redis启动之初会读取该文件重新构造数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。  
### AOF持久化流程  
客户端的请求写命令会被append追加到AOF缓冲区内  
AOF缓冲区会根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中  
AOF文件大小超过重写策略或手动重写时，会对AOF文件进行重写（rewrite），压缩AOF文件容量  
redis服务器重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的  
AOF和RDB同时开启，系统默认取AOF的数据（数据不会存在丢失）  
### 配置
可以在 redis.conf 文件中对AOF进行配置  
```
appendonly no # 是否开启AOF，yes：开启，no：不开启，默认为no
appendfilename "appendonly.aof" # aof文件名称，默认为appendonly.aof
dir ./ # aof文件所在目录，默认./，表示执行启动命令时所在的目录，比如我们在/opt目录中，去执行
redis-server /etc/redis.conf 来启动redis，那么dir此时就是/opt目录
appendfsync everysec  设置备份频率
appendfsync always：每次写入立即同步
appendfsync everysec：每秒同步
appendfsync no：不主动同步
no-appendfsync-on-rewrite：重写时，不会执行appendfsync操作
```
### rewrite压缩（AOF文件压缩）  
AOF采用文件追加方式，文件会越来越大，为了避免出现此情况，新增了重写机制，当AOF文件的大小超过锁审定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以用命令bgrewriteaof触发重写。  
重写原理，如何实现重写？  
AOF文件持续增长而过大时，会fork出一条新进程来将文件重写（也是先写临时文件，最后在rename替换旧文件），redis4.0版本后的重写，是指就把rdb的快照，以二进制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作  

重写时机：

bgrewriteaof：手动触发重写  
auto-aof-rewrite-percentage：设置重写基准值  
设置重写的基准值，默认100，当文件达到100%时开始重写（文件是原来重写后文件的2倍时重写）。  
auto-aof-rewrite-min-size：设置重写基准值  
设置重写的基准值，默认64MB，AOF文件大小超过这个值开始重写。  

重写流程：  

1、手动执行 bgrewriteaof 命令触发重写，判断是否当前有bgfsave或bgrewriteaof在运行，如果有，则等待该命令结束后再继续执行  
2、主进程fork出子进程执行重写操作，保证主进程不会阻塞  
3、子进程遍历redis内存中的数据到临时文件，客户端的写请求同时写入aof_buf缓冲区和aof_rewrite_buf重写缓冲区保证原AOF文件完整性以及新AOF文件生成期间的新的数据修改动作不会丢失  
4、子进程写完新的AOF文件后，向主进程发送信号，父进程更新统计信息  
5、主进程把aof_rewrite_buf中的数据写入到新的AOF文件  
6、使用新的AOF文件覆盖旧的AOF文件，完成AOF重写  

### 优势
备份机制更稳健，丢失数据概率更低  
可读的日志文本，通过操作AOF文件，可以处理误操作  
### 劣势
比RDB占用更多的磁盘空间  
恢复备份速度要慢  
每次读写都同步的话，有一定的性能压力  
存在个别bug，造成不能恢复  
## 用哪个好？
官方推荐2个都启用。  
如果对数据不敏感，可以单独用RDB。  
不建议单独使用AOF，因为可能会出现BUG。  
如果只是做纯内存缓存，可以都不用。  


