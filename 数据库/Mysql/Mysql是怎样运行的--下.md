@[toc]
# Mysql是怎样运行的--下

# 查询优化

## explain

## optimizer_trace

查看是否开启  
```
SHOW VARIABLES LIKE 'optimizer_trace';
```
开启功能  
```
SET optimizer_trace="enabled=on";
```
查看具体的优化过程  
```
SELECT * FROM information_schema.OPTIMIZER_TRACE;
```
这个OPTIMIZER_TRACE表有4个列，分别是：    

* QUERY：表示我们的查询语句。    
* TRACE：表示优化过程的JSON格式文本。    
* MISSING_BYTES_BEYOND_MAX_MEM_SIZE：由于优化过程可能会输出很多，如果超过某个限制时，多余的文本将不会被显示，这个字段展示了被忽略的文本字节数。   
* INSUFFICIENT_PRIVILEGES：表示是否没有权限查看优化过程，默认值是0，只有某些特殊情况下才会是1，我们暂时不关心这个字段的值。  
完整的使用步骤如下  
```
1. 打开optimizer trace功能 (默认情况下它是关闭的):  
SET optimizer_trace="enabled=on";  
2. 这里输入你自己的查询语句  
SELECT ;  
3. 从OPTIMIZER_TRACE表中查看上一个查询的优化过程  
SELECT * FROM information_schema.OPTIMIZER_TRACE;  
4. 可能你还要观察其他语句执行的优化过程，重复上面的第2、3步  
5. 当你停止查看语句的优化过程时，把optimizer trace功能关闭  
SET optimizer_trace="enabled=off";  
```

化过程大致分为了三个阶段：  

prepare阶段  
optimize阶段  
execute阶段  
 
于单表查询来说，我们主要关注optimize阶段的"rows_estimation"这个过程  
对于多表连接查询来说，我们更多需要关注"considered_execution_plans"这个过程  

# InnoDB的Buffer Pool（缓冲池）  
InnoDB的设计者为了存磁盘中的页，在MySQL服务器启动的时候就向操作系统申请了一片连续的内存，他们给这片内存起了个名，叫做Buffer Pool。  
默认情况下Buffer Pool只有128M大小。当然如果你嫌弃这个128M太大或者太小，可以在启动服务器的时候配置innodb_buffer_pool_size参数的值，是以字节为单位的  
需要注意的是，Buffer Pool也不能太小，最小值为5M(当小于该值时会自动设置成5M)。  
## Buffer Pool的存储结构  
我们就把每个页对应的控制信息占用的一块内存称为一个控制块吧，控制块和缓存页是一一对应的，它们都被存放到 Buffer Pool 中，其中控制块被存放到 Buffer Pool 的前面，缓存页被存放到 Buffer Pool 后边，所以整个Buffer Pool对应的内存空间看起来就是这样的：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/22f01f0ab06b44a984e85a070c27e65c.png)


### 空闲页存储--free链表  
把所有空闲的缓存页对应的控制块作为一个节点放到一个链表中，这个链表也可以被称作**free链表**  
为了管理好这个free链表，特意为这个链表定义了一个基节点，里边儿包含着链表的头节点地址，尾节点地址，以及当前链表中节点的数量等信息。这里需要注意的是，链表的基节点占用的内存空间并不包含在为Buffer Pool申请的一大片连续内存空间之内，而是单独申请的一块内存空间。  

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/20dd8bbbc6fb4038b109b95e57e3b94e.png)



有了这个free链表之后事儿就好办了，每当需要从磁盘中加载一个页到Buffer Pool中时，就从free链表中取一个空闲的缓存页，并且把该缓存页对应的控制块的信息填上（就是该页所在的表空间、页号之类的信息），然后把该缓存页对应的free链表节点从链表中移除，表示该缓存页已经被使用了  
### 脏页（修改后的数据）存储--flush链表

我们修改了Buffer Pool中某个缓存页的数据，那它就和磁盘上的页不一致了，这样的缓存页也被称为脏页，我们创建一个存储脏页的链表，凡是修改过的缓存页对应的控制块都会作为一个节点加入到一个链表中，因为这个链表节点对应的缓存页都是需要被刷新到磁盘上的，所以也叫flush链表，数据结构与free链表一样  

## 使用Buffer Pool

1、读取时的Hash处理：  
以用表空间号 + 页号作为key，缓存页作为value创建一个哈希表，在需要访问某个页的数据时，先从哈希表中根据表空间号 + 页号看看有没有对应的缓存页，如果有，直接使用该缓存页就好，如果没有，那就从free链表中选一个空闲的缓存页，然后把磁盘中对应的页加载到该缓存页的位置。  
2、修改数据时脏页处理：  
从flush链表中异步同步脏页数据，这种刷新页面的方式被称之为BUF_FLUSH_LIST  
从LRU链表的冷数据中刷新一部分页面到磁盘，方式被称之为BUF_FLUSH_LRU  
3、查看状态  
SHOW ENGINE INNODB STATUS语句来查看关于InnoDB存储引擎运行过程中的一些状态信息，其中就包括Buffer Pool的一些信息  

## LRU链表的管理
缓存的页占用的内存大小超过了Buffer Pool大小要进行删除不常用的页  

简单设计：  
只要我们使用到某个缓存页，就把该缓存页调整到LRU链表的头部，这样LRU链表尾部就是最近最少使用的缓存页喽  
但是有两种情况会导致缓存命中大大降低：  
1、预读功能  
读取一个区，另一个区也会预先读取BufferPool  
读取一个区的连续13个页，整个区也会读取到BufferPool  
2、全表扫描  

这两种情况可能并不是常用数据但是会导致很多BufferPool的数据被删掉  

设计InnoDB的大佬把这个LRU链表按照一定比例分成两截，分别是：  

一部分存储使用频率非常高的缓存页，所以这一部分链表也叫做热数据，或者称**young区域**。  
另一部分存储使用频率不是很高的缓存页，所以这一部分链表也叫做冷数据，或者称**old区域**。  
我们可以通过查看系统变量innodb_old_blocks_pct的值来确定old区域在LRU链表中所占的比例  
```
 SHOW VARIABLES LIKE 'innodb_old_blocks_pct';
```
默认情况下，old区域在LRU链表中所占的比例是37%  

针对上面两种情况：  
1、当磁盘上的某个页面在初次加载到Buffer Pool中的某个缓存页时，该缓存页对应的控制块会被放到old区域的头部。这样针对预读到Buffer Pool却不进行后续访问的页面就会被逐渐从old区域逐出，而不会影响young区域中被使用比较频繁的缓存页。
2、针对全表扫描时，在进行全表扫描时，虽然首次被加载到Buffer Pool的页被放到了old区域的头部，但是后续会被马上访问到，每次进行访问的时候又会把该页放到young区域的头部  
在对某个处在old区域的缓存页进行第一次访问时就在它对应的控制块中记录下来这个访问时间，如果后续的访问时间与第一次访问的时间在某个时间间隔内，那么该页面就不会被从old区域移动到young区域的头部，否则将它移动到young区域的头部。  
时间间隔由一下参数决定：  
```
SHOW VARIABLES LIKE 'innodb_old_blocks_time';
```
更进一步优化LRU链表：只有被访问的缓存页位于young区域的1/4的后边，才会被移动到LRU链表头部，这样就可以降低调整LRU链表的频率，从而提升性能。
还有其他的需要补充  

# 事务
## ACID  
Atomicity：要不全部成功，要么全部失败. 事务是原子的，要么全部执行成功，要么全部失败。如果事务中的任何一部分失败，系统应该回滚到事务开始前的状态，以保持数据库的一致性。  
Isolation：现场A对线程B不可见, 事务的执行应该相互隔离，一个事务的执行不应该影响其他事务的执行。这是通过隔离级别来控制的，如读未提交、读提交、可重复读和串行化。  
Consistency：事务执行前后数据库应该保持一致性。这包括应用事务中定义的业务规则和完整性约束。例如，数据库中的主键、外键和其他约束条件在事务执行前后都应该得到满足。  
Durability：一旦事务被提交，对数据库的更改应该是永久性的，并且在数据库发生故障或系统崩溃时也应该能够恢复。  

## 事务的状态
活动的（active）  
部分提交  
失败的  
终止的  
提交的  

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/537e5c97ac20468bb4b6217affefedcf.png)


## 事务语法
1、开启事务：BEGIN [WORK];  
BEGIN语句代表开启一个事务，后边的单词WORK可有可无。开启事务后，就可以继续写若干条语句，这些语句都属于刚刚开启的这个事务。  
或者：START TRANSACTION;  
START TRANSACTION语句和BEGIN语句有着相同的功效，都标志着开启一个事务  
READ ONLY：标识当前事务是一个只读事务，也就是属于该事务的数据库操作只能读取数据，而不能修改数据。  
READ WRITE：标识当前事务是一个读写事务，也就是属于该事务的数据库操作既可以读取数据，也可以修改数据。  
WITH CONSISTENT SNAPSHOT：启动一致性读  
可以这样写：  
```
START TRANSACTION READ ONLY, WITH CONSISTENT SNAPSHOT;//开启一个只读事务和一致性读
```
```
START TRANSACTION READ WRITE, WITH CONSISTENT SNAPSHOT //开启一个读写事务和一致性读
```
2、提交事务:COMMIT [WORK]  
3、回滚事务：ROLLBACK [WORK]  
4、自动提交  
```
 SHOW VARIABLES LIKE 'autocommit';
```
5、隐式提交  
显示的提交：当我们使用START TRANSACTION或者BEGIN语句开启了一个事务，或者把系统变量autocommit的值设置为OFF时，事务就不会进行自动提交。  
* 定义或修改数据库对象的数据定义语言（Data definition language，缩写为：DDL）。当我们使用CREATE、ALTER、DROP等语句去修改这些所谓的数据库对象时  
* 隐式使用或修改mysql数据库中的表  
* 事务控制或关于锁定的语句，使用LOCK TABLES、UNLOCK TABLES等关于锁定的语句  
* 加载数据的语句，比如我们使用LOAD DATA语句来批量往数据库中导入数据时  
* 关于MySQL复制的一些语句，START SLAVE、STOP SLAVE、RESET SLAVE、CHANGE MASTER TO等语句  
* 其它的一些语句，ANALYZE TABLE、CACHE INDEX、CHECK TABLE、FLUSH、 LOAD INDEX INTO CACHE、OPTIMIZE TABLE、REPAIR TABLE、RESET  
6、保存点  
定义保存点  
```
SAVEPOINT 保存点名称;  
```
回滚到某个保存点时  
```
ROLLBACK [WORK] TO [SAVEPOINT] 保存点名称; 
```
# 日志

## redo日志  
redo日志是InnoDB存储引擎的一部分，用于保证事务的持久性和恢复能力。  
在InnoDB中，redo日志是以物理方式记录的，而不是直接写入磁盘的数据文件。它包含了对数据库所做的所有修改操作，包括插入、更新和删除等。redo日志的作用是在发生故障或崩溃时，通过重做日志中的操作来恢复数据库到最新的一致状态。  
redo日志存储在InnoDB的日志文件中，通常称为redo日志文件或者事务日志文件。在MySQL的数据目录下，可以找到以ib_logfile开头的文件，例如ib_logfile0、ib_logfile1等。这些文件就是InnoDB的redo日志文件。  
redo日志文件是循环写入的，当写满一个日志文件后，会继续写入下一个日志文件。当所有的日志文件都写满后，InnoDB会将最旧的日志文件复用，覆盖其中的内容。因此，redo日志文件的大小和数量可以通过配置参数进行调整，以适应不同的需求。  

### 简述  
真正访问页面之前，需要把在磁盘上的页缓存到内存中的Buffer Pool之后才可以访问，如果一个事务对于数据做了修改，也是现在BufferPool中生效，后面再刷到磁盘中，但是如果还没有刷到磁盘中导致在事务提交之后出现了事故，那么可能会产生一致性问题。
因此提出了redo日志，也叫重写日志。原理是在提交事务之前把修改了哪些东西记录一下，系统奔溃重启时需要按照记录的步骤重新更新数据页即可。  
redo日志有两个优势：  
redo日志占用的空间非常小  
redo日志是顺序写入磁盘的  

redo日志格式：  
* type：该条redo日志的类型。  
* space ID：表空间ID。  
* page number：页号。  
* data：该条redo日志的具体内容。  
### 恢复时保证原子性  
重启进行恢复时也要保证原子性，因此在redo日志中必须要以一个类型为MLOG_MULTI_REC_END结尾，在系统奔溃重启进行恢复时，只有当解析到类型为MLOG_MULTI_REC_END的redo日志，才认为解析到了一组完整的redo日志，才会进行恢复。否则的话直接放弃前面解析到的redo日志。  
当然，如果只有一条语句，那么将会以type字段的第一个比特位为1，代表该需要保证原子性的操作只产生了单一的一条redo日志，不需要关注MLOG_MULTI_REC_END，否则表示该需要保证原子性的操作产生了一系列的redo日志。  
### 写入过程  
通过mtr生成的redo日志都放在了大小为512字节的页中，称为block。  

结构如下：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/ea7cfc4691c9432ca6e8d9e2ffdbbd4e.png)

在这里插入图片描述  

写入时会把block写入缓冲区，这个缓冲区叫做log buffer（redo日志缓冲区），我们可以通过启动参数innodb_log_buffer_size来指定log buffer的大小，默认是16M，刷入到缓存就会找时机将缓存数据刷到磁盘中。  
redo日志刷盘时机：  
1、log buffer空间不足时：占据大约1半就会刷   
2、事务提交时  
3、后台线程不停的刷刷刷： 后台有一个线程，大约每秒都会刷新一次log buffer中的redo日志到磁盘。  
4、正常关闭服务器时  
5、做的checkpoint时  
6、其他  

### redo日志设置参数  
SHOW VARIABLES LIKE 'datadir'查看  
下默认有两个名为ib_logfile0和ib_logfile1的文件，log buffer中的日志默认情况下就是刷新到这两个磁盘文件中。  
参数调节：  
innodb_log_group_home_dir  
  该参数指定了redo日志文件所在的目录，默认值就是当前的数据目录。  
innodb_log_file_size  
      该参数指定了每个redo日志文件的大小，在MySQL 5.7.21这个版本中的默认值为48MB  
innodb_log_files_in_group  
      该参数指定redo日志文件的个数，默认值为2，最大值为100。  

没看完[redo日志（下）](https://relph1119.github.io/mysql-learning-notes/#/mysql/21-说过的话就一定要办到-redo日志（下）)

## undo日志  
### 描述  
我们说过事务需要保证原子性，也就是事务中的操作要么全部完成，要么什么也不做。但是偏偏有时候事务执行到一半会出现一些情况，比如：  
情况一：事务执行过程中可能遇到各种错误，比如服务器本身的错误，操作系统错误，甚至是突然断电导致的错误。  
情况二：程序员可以在事务执行过程中手动输入ROLLBACK语句结束当前的事务的执行。  

我们要对一条记录做改动时（这里的改动可以指INSERT、DELETE、UPDATE），都需要留一手 —— 把回滚时所需的东西都给记下来,把这些为了回滚而记录的这些数据称之为撤销日志，英文名为undo log，称之为**undo日志**。  

InnoDB存储引擎在实际进行增、删、改一条记录时，都需要先把对应的undo日志记下来。一般每对一条记录做一次改动，就对应着一条undo日志，但在某些更新记录的操作中，也可能会对应着2条undo日志。  
一个事务在执行过程中可能新增、删除、更新若干条记录，也就是说需要记录很多条对应的undo日志，这些undo日志会被从0开始编号，也就是说根据生成的顺序分别被称为第0号undo日志、第1号undo日志、...、第n号undo日志等，这个编号也被称之为**undo no**。 这些undo日志是被记录到类型为FIL_PAGE_UNDO_LOG的页面中，这些页面可以从系统表空间中分配，也可以从一种专门存放undo日志的表空间，也就是所谓的**undo tablespace**中分配。  
### 事务id  
事务id是怎么生成的？  
1. 服务器会在内存中维护一个全局变量，每当需要为某个事务分配一个事务id时，就会把该变量的值当作事务id分配给该事务，并且把该变量自增1 
2. 每当这个变量的值为256的倍数时，就会将该变量的值刷新到系统表空间的页号为5的页面中一个称之为Max Trx ID的属性处，这个属性占用8个字节的存储空间。  
3. 当系统下一次重新启动时，会将上面提到的Max Trx ID属性加载到内存中，将该值加上256之后赋值给我们前面提到的全局变量（因为在上次关机时该全局变量的值可能大于Max Trx ID属性值）。  
### trx_id  
聚簇索引的记录除了会保存完整的用户数据以外，而且还会自动添加名为trx_id、roll_pointer的隐藏列，如果用户没有在表中定义主键以及UNIQUE键，还会自动添加一个名为row_id的隐藏列。  
就是某个对这个聚簇索引记录做改动的语句所在的事务对应的事务id而已，此处的改动可以是INSERT、DELETE、UPDATE操作  

### 查看Table ID
例如：  
```
 SELECT * FROM information_schema.innodb_sys_tables;
```
### 增删改对应的undo日志  
#### INSERT操作对应的undo日志  
InnoDB设计了一个类型为TRX_UNDO_INSERT_REC的undo日志，它的完整结构如下图所示：  

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/05b3a839cb4e43e9bfae0e341907f80b.png)



需要了解几点：  
* undo no在一个事务中是从0开始递增的，也就是说只要事务没提交，每生成一条undo日志，那么该条日志的undo no就增1。  
* 如果记录中的主键只包含一个列，那么在类型为TRX_UNDO_INSERT_REC的undo日志中只需要把该列占用的存储空间大小和真实值记录下来，如果记录中的主键包含多个列，那么每个列占用的存储空间大小和对应的真实值都需要记录下来（图中的len就代表列占用的存储空间大小，value就代表列的真实值）。  

实例：  
插入两条数据  
```
BEGIN;  # 显式开启一个事务，假设该事务的id为100

# 插入两条记录  
INSERT INTO undo_demo(id, key1, col)   
    VALUES (1, 'AWM', '狙击枪'), (2, 'M416', '步枪')
```
记录两条undo日志  
第一条undo日志的undo no为0，记录主键占用的存储空间长度为4，真实值为1。画一个示意图就是这样：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/70659520a2134785aa5ce24ad9a93af9.png)


第二条undo日志的undo no为1，记录主键占用的存储空间长度为4，真实值为2。画一个示意图就是这样（与第一条undo日志对比，undo no和主键各列信息有不同）：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/dc28ced1ff1849e1ac5267484660ec19.png)



#### DELETE操作对应的undo日志  
首先了解一下delete mark操作：  
Delete mark（删除标记）是InnoDB存储引擎中的一种操作，用于标记数据行被删除的状态。它是InnoDB实现MVCC（多版本并发控制）的关键机制之一。  
在InnoDB中，当执行DELETE操作时，并不会立即删除数据行，而是通过设置一个特殊的删除标记来标记该行为已删除状态。这个删除标记称为delete mark。  
Delete mark的作用是在事务的隔离级别为可重复读（REPEATABLE READ）或更高级别时，保证在事务执行期间，其他事务不会读取到已删除的数据行  
当一个事务需要读取数据时，InnoDB会根据事务的隔离级别判断是否需要忽略已经被删除的数据行。如果事务的隔离级别为可重复读或更高级别，InnoDB会使用delete mark来判断数据行是否可见。如果数据行被标记为已删除，则InnoDB会忽略这些数据行，不将其包含在查询结果中。  
需要注意的是，delete mark并不是永久性的删除数据行，而是在事务提交后，由后台的purge线程负责清理已删除的数据行。purge线程会根据事务的提交时间和MVCC的版本链来判断哪些数据行可以被永久删除。  
总结起来，delete mark是InnoDB存储引擎中的一种操作，用于标记已删除的数据行。它是实现MVCC的关键机制之一，用于保证在事务的隔离级别为可重复读或更高级别时，其他事务不会读取到已删除的数据行。删除标记并不是永久性的删除数据行，而是在事务提交后由purge线程清理。  


插入到页面中的记录会根据记录头信息中的next_record属性组成一个单向链表，我们把这个链表称之为正常记录链表    
被删除的记录其实也会根据记录头信息中的next_record属性组成一个链表，只不过这个链表中的记录占用的存储空间可以被重新利用，所以也称这个链表为垃圾链表，Page Header部分有一个称之为PAGE_FREE的属性，它指向由被删除记录组成的垃圾链表中的头节点。

删除流程如下：  
1、仅仅将记录的delete_mask标识位设置为1，其他的不做修改  
正常记录链表中的最后一条记录的delete_mask值被设置为1，但是并没有被加入到垃圾链表。也就是此时记录处于一个中间状态  
在删除语句所在的事务提交之前，被删除的记录一直都处于这种所谓的中间状态。  
2、当该删除语句所在的事务提交之后，会有专门的线程后来真正的把记录删除掉。  
所谓真正的删除就是把该记录从正常记录链表中移除，并且加入到垃圾链表中，然后还要调整一些页面的其他信息，比如页面中的用户记录数量PAGE_N_RECS、上次插入记录的位置PAGE_LAST_INSERT、垃圾链表头节点的指针PAGE_FREE、页面中可重用的字节数量PAGE_GARBAGE、还有页目录的一些信息等等。设计InnoDB的大佬把这个阶段称之为purge。  

undo日志的操作：  
1、在对一条记录进行delete mark操作前，需要把该记录的旧的trx_id和roll_pointer隐藏列的值都给记到对应的undo日志中来  
```
# 删除一条记录    
DELETE FROM undo_demo WHERE id = 1; 
```
  
这个delete mark操作对应的undo日志的结构就是这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/7e0ce7daf55c4809b42166b0ab96598b.png)


* 这条undo日志是id为100的事务中产生的第3条undo日志，所以它对应的undo no就是2。  
* 在对记录做delete mark操作时，记录的trx_id隐藏列的值是100，本次事务的id，所以把100填入old trx_id属性中。然后把记录的roll_pointer隐藏列的值取出来，填入old roll_pointer属性中，这样就可以通过old roll_pointer属性值找到最近一次对该记录做改动时产生的undo日志。  
* 由于undo_demo表中有2个索引：一个是聚簇索引，一个是二级索引idx_key1。只要是包含在索引中的列，那么这个列在记录中的位置（pos），占用存储空间大小（len）和实际值（value）就需要存储到undo日志中。  
* 对于主键来说，只包含一个id列，存储到undo日志中的相关信息分别是：  
  pos：id列是主键，也就是在记录的第一个列，它对应的pos值为0。pos占用1个字节来存储。  
  len：id列的类型为INT，占用4个字节，所以len的值为4。len占用1个字节来存储。  
  value：在被删除的记录中id列的值为1，也就是value的值为1。value占用4个字节来存储。  
  所以对于id列来说，最终存储的结果就是<0, 4, 1>，存储这些信息占用的存储空间大小为1 + 1 + 4 = 6个字节。
  
#### UPDATE操作对应的undo日志
 在执行UPDATE语句时，InnoDB对更新主键和不更新主键这两种情况有截然不同的处理方案。  
不更新主键的情况  
1、就地更新（in-place update）  
2、先删除掉旧记录，再插入新记录  
我们这里所说的删除并不是delete mark操作，而是真正的删除掉，也就是把这条记录从正常记录链表中移除并加入到垃圾链表中  
更新主键的情况  
1、将旧记录进行delete mark操作  
2、根据更新后各列的值创建一条新记录，并将其插入到聚簇索引中（需重新定位插入的位置）。   
## 存储undo日志
undo日志，默认存储在数据目录下的系统表空间文件中。系统表空间文件的默认命名为ibdata1。  
从MySQL 5.7.6版本开始，InnoDB引入了多表空间的概念，可以将Undo表空间存储在独立的文件中，而不再与系统表空间混合。这样可以更好地管理Undo表空间的大小和回收。  
表空间的具体数据都是以页的形式存储的，undo日志也不例外，被称为FIL_PAGE_UNDO_LOG的页  
### 状态查看
执行以下查询语句，查看InnoDB的undo日志信息：  

```
SHOW ENGINE INNODB STATUS\G
```
找到查询结果中的"Innodb Status"部分，并查找到"TRANSACTIONS"标签下的"History list length"。  

例如：  

```
Trx id counter 123456
Purge done for trx's n: 0:123456
Undo log entries 10 // 这里是undo日志的长度
History list length 1000 这里是undo历史列表的长度
```
### Undo页面链表
 因为一个事务可能包含多个语句，而且一个语句可能对若干条记录进行改动，而对每条记录进行改动前，都需要记录1条或2条的undo日志，所以在一个事务执行过程中可能产生很多undo日志，这些日志可能一个页面放不下，需要放到多个页面中，这些页面就通过我们上面介绍的TRX_UNDO_PAGE_NODE属性连成了链表  
#### 单个事务中的Undo页面链表
在一个事务执行过程中，可能混着执行INSERT、DELETE、UPDATE语句，也就意味着会产生不同类型的undo日志。但是我们前面又强调过，同一个Undo页面要么只存储TRX_UNDO_INSERT大类的undo日志，要么只存储TRX_UNDO_UPDATE大类的undo日志，反正不能混着存，所以在一个事务执行过程中就可能需要2个Undo页面的链表，一个称之为insert undo链表，另一个称之为update undo链表。  

另外，设计InnoDB的大佬规定对普通表和临时表的记录改动时产生的undo日志要分别记录（我们稍后阐释为什么这么做），所以在一个事务中最多有4个以Undo页面为节点组成的链。  
总结：  
* 刚刚开启事务时，一个Undo页面链表也不分配。  
* 当事务执行过程中向普通表中插入记录或者执行更新记录主键的操作之后，就会为其分配一个普通表的insert undo链表。  
* 当事务执行过程中删除或者更新了普通表中的记录之后，就会为其分配一个普通表的update undo链表。  
* 当事务执行过程中向临时表中插入记录或者执行更新记录主键的操作之后，就会为其分配一个临时表的insert undo链表。  
* 当事务执行过程中删除或者更新了临时表中的记录之后，就会为其分配一个临时表的update undo链表。  
  总结一句就是：**按需分配，什么时候需要什么时候再分配，不需要就不分配**。  
#### 多个事务中的Undo页面链表  
 为了尽可能提高undo日志的写入效率，不同事务执行过程中产生的undo日志需要被写入到不同的Undo页面链表中。  
### undo日志具体写入过程  
#### 基本流程  
存储结构是也采用了段，称为：Segment Header，每一个Undo页面链表都对应着一个段，称之为Undo Log Segment  
段的结构为：  
* Space ID of the INODE Entry：INODE Entry结构所在的表空间ID。  
* Page Number of the INODE Entry：INODE Entry结构所在的页面页号。  
* Byte Offset of the INODE Ent：INODE Entry结构在该页面中的偏移量  

一个事务在向Undo页面中写入undo日志时的方式是十分简单暴力的，就是直接往里怼，写完一条紧接着写另一条，各条undo日志之间是亲密无间的。写完一个Undo页面后，再从段里申请一个新页面，然后把这个页面插入到Undo页面链表中，继续往这个新申请的页面中写。

Undo页面链表的第一个页面在真正写入undo日志前，其实都会被填充Undo Page Header、Undo Log Segment Header、Undo Log Header这3个部分  
#### 重用Undo页面
重用情形：  
1、该链表中只包含一个Undo页面。  
2、insert undo链表。  
insert undo链表中只存储类型为TRX_UNDO_INSERT_REC的undo日志，这种类型的undo日志在事务提交之后就没用了，就可以被清除掉。所以在某个事务提交后，重用这个事务的insert undo链表（这个链表中只有一个页面）时，可以直接把之前事务写入的一组undo日志覆盖掉，从头开始写入新事务的一组undo日志  
3、update undo链表  
在一个事务提交后，它的update undo链表中的undo日志也不能立即删除掉（这些日志用于MVCC）。所以如果之后的事务想重用update undo链表时，就不能覆盖之前事务写入的undo日志。这样就相当于在同一个Undo页面中写入了多组的undo日志  
## 回滚段
[undo日志（下）](https://relph1119.github.io/mysql-learning-notes/#/mysql/23-后悔了怎么办-undo日志（下）)

# 事务隔离级别与MVCC-读  
## 事务并发的三个问题  
* 脏读：读取未提交数据导致的  
  原因：一个事务修改了另一个未提交事务修改过的数据  

* 不可重复度：重复读取同一个数据两次结果不一样  
   原因：一个事务只能读到另一个已经提交的事务修改过的数据，并且其他事务每对该数据进行一次修改并提交后，该事务都能查询得到最新值  

* 幻读：范围查找时多了数据  
  原因：个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次按照该条件查询时，能把另一个事务插入的记录也读出来  

严重性：脏写 > 脏读 > 不可重复读 > 幻读  
## 事务四种隔离级别
设立一些隔离级别，隔离级别越低，越严重的问题就越可能发生  
* READ UNCOMMITTED：未提交读。可能发生脏读、不可重复读和幻读问题。  
* READ COMMITTED：已提交读，可能发生不可重复读和幻读问题，但是不可以发生脏读问题。  
* REPEATABLE READ：可重复读，可能发生幻读问题，但是不可以发生脏读和不可重复读的问题。  
* SERIALIZABLE：可串行化，各种问题都不可以发生。  

Oracle就只支持READ COMMITTED和SERIALIZABLE  
设置方法：  
```
SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL level;
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;

可选：
level: {
     REPEATABLE READ
   | READ COMMITTED
   | READ UNCOMMITTED
   | SERIALIZABLE
}
```
## MVCC原理  
### 版本链  
对于使用InnoDB存储引擎的表来说，它的聚簇索引记录中都包含两个必要的隐藏列（row_id并不是必要的，我们创建的表中有主键或者非NULL的UNIQUE键时都不会包含row_id列）：  
* trx_id：每次一个事务对某条聚簇索引记录进行改动时，都会把该事务的事务id赋值给trx_id隐藏列。  
* roll_pointer：每次对某条聚簇索引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。  
随着更新次数的增多，所有的版本都会被roll_pointer属性连接成一个链表，我们把这个链表称之为版本链，版本链的头节点就是当前记录最新的值。另外，每个版本中还包含生成该版本时对应的事务id  
### ReadView  
1、对于使用READ UNCOMMITTED隔离级别的事务来说，由于可以读到未提交事务修改过的记录，所以直接读取记录的最新版本就好了  
2、对于使用SERIALIZABLE隔离级别的事务来说，设计InnoDB规定使用加锁的方式来访问记录  
3、对于使用READ COMMITTED和REPEATABLE READ隔离级别的事务来说，都必须保证读到已经提交了的事务修改过的记录，也就是说假如另一个事务已经修改了记录但是尚未提交，是不能直接读取最新版本的记录的  
为了完成上述的不同阶段的读取要求，需**要判断一下版本链中的哪个版本是当前事务可见的**，提出了一个ReadView的概念，这个ReadView中主要包含4个比较重要的内容：  
* m_ids：表示在生成ReadView时当前系统中活跃的读写事务的事务id列表。  
* min_trx_id：表示在生成ReadView时当前系统中活跃的读写事务中最小的事务id，也就是m_ids中的最小值。  
* max_trx_id：表示生成ReadView时系统中应该分配给下一个事务的id值。  
* creator_trx_id：表示生成该ReadView的事务的事务id。  
这四个属性的作用如下：  
* m_ids：用于存储当前活跃的读写事务的id，这些事务可能正在修改数据，因此需要排除在当前事务之外。  
* min_trx_id：用于确定当前事务能够看到的最早的版本，即所有事务id大于min_trx_id的事务所做的修改对于当前事务都是不可见的。  
* max_trx_id：用于确定当前事务的id，如果一个事务的id小于max_trx_id，那么这个事务在当前事务开始之前就已经提交了，所以对当前事务来说是可见的。  
* creator_trx_id就是用来标识这个可见版本是由哪个事务创建的。如果这个事务已经提交了，那么creator_trx_id就是这个事务的ID；如果这个事务还没有提交，那么creator_trx_id就是这个事务的ID加上一个特殊的值，表示这个版本是由一个未提交的事务创建的。
当一个事务要访问某个数据时，它会检查这个数据的版本链，只需要按照下面的步骤判断记录的某个版本是否可见：  
* 如果被访问版本的trx_id属性值与ReadView中的creator_trx_id值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。  
* 如果被访问版本的trx_id属性值小于ReadView中的min_trx_id值，表明生成该版本的事务在当前事务生成ReadView前已经提交，所以该版本可以被当前事务访问。  
* 如果被访问版本的trx_id属性值大于ReadView中的max_trx_id值，表明生成该版本的事务在当前事务生成ReadView后才开启，所以该版本不可以被当前事务访问。  
* 如果被访问版本的trx_id属性值在ReadView的min_trx_id和max_trx_id之间，那就需要判断一下trx_id属性值是不是在m_ids列表中，如果在，说明创建ReadView时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建ReadView时生成该版本的事务已经被提交，该版本可以被访问。  
如果某个版本的数据对当前事务不可见的话，那就顺着版本链找到下一个版本的数据，继续按照上面的步骤判断可见性，依此类推，直到版本链中的最后一个版本。如果最后一个版本也不可见的话，那么就意味着该条记录对该事务完全不可见，查询结果就不包含该记录。
### READ COMMITTED和REPEATABLE READ隔离级别区别  
MySQL中，READ COMMITTED和REPEATABLE READ隔离级别的的一个非常大的区别就是它们生成ReadView的时机不同。  

在READ COMMITTED隔离级别中，ReadView是在每个SQL语句执行前都会生成的。这意味着在事务中的同一个查询可能会返回不同的结果，因为每次查询都会看到最新的数据。  

而在REPEATABLE READ隔离级别中，ReadView是在事务开始时就生成的，并且会一直保持到事务结束。这就保证了在同一个事务中，所有的查询都将看到同样版本的数据。  

例如：  
例如，考虑以下两个事务：  

事务A：  
```
START TRANSACTION;
SELECT * FROM table WHERE id = 1; -- 返回空集
INSERT INTO table (id, name) VALUES (1, 'John');
COMMIT;
```
事务B：  
```
START TRANSACTION;
SELECT * FROM table WHERE id = 1; -- 可能返回空集，也可能返回(1, 'John')
COMMIT;
```
如果这两个事务在READ COMMITTED隔离级别下执行，那么事务B可能会在事务A提交后才看到新插入的数据。但是，如果这两个事务在REPEATABLE READ隔离级别下执行，那么事务B将始终看不到新插入的数据，因为它的ReadView是在事务开始时生成的。  

### 小结  
所谓的MVCC（Multi-Version Concurrency Control ，多版本并发控制）指的就是在使用READ COMMITTD、REPEATABLE READ这两种隔离级别的事务在执行普通的SEELCT操作时访问记录的版本链的过程，这样子可以使不同事务的读-写、写-读操作并发执行，从而提升系统性能。READ COMMITTD、REPEATABLE READ这两个隔离级别的一个很大不同就是：生成ReadView的时机不同，READ COMMITTD在每一次进行普通SELECT操作前都会生成一个ReadView，而REPEATABLE READ只在第一次进行普通SELECT操作前生成一个ReadView，之后的查询操作都重复使用这个ReadView就好了  

# 锁-写
 怎么解决脏读、不可重复读、幻读这些问题呢？其实有两种可选的解决方案：
方案一：读操作利用多版本并发控制（MVCC），写操作进行加锁。
方案二：读、写操作都采用加锁的方式。
## 一致性读
事务利用MVCC进行的读取操作称之为一致性读，或者一致性无锁读，有的地方也称之为快照读。所有普通的SELECT语句（plain SELECT）在READ COMMITTED、REPEATABLE READ隔离级别下都算是一致性读。
 一致性读并不会对表中的任何记录做加锁操作，其他事务可以自由的对表中的记录做改动。
## 锁定读（Locking Reads）
### 共享锁和独占锁
共享锁，英文名：Shared  Locks，简称S锁。在事务要读取一条记录时，需要先获取该记录的S锁。也称为读锁
独占锁，也常称排他锁，英文名：Exclusive Locks，简称X锁。在事务要改动一条记录时，需要先获取该记录的X锁，也称为写锁

两者兼容性如下：
```
读读兼容
读写不兼容
写写不兼容
```
总而言之：只有两个线程都是读锁能同时读，其他都不行

**锁定读的语句**
```
SELECT ... LOCK IN SHARE MODE; //对读取的记录加S锁
SELECT ... FOR UPDATE; //对读取的记录加X锁
```
**DELETE**
对一条记录做DELETE操作的过程其实是先在B+树中定位到这条记录的位置，然后获取一下这条记录的X锁，然后再执行delete mark操作
**UPDATE**
在对一条记录做UPDATE操作时分为三种情况：
如果未修改该记录的键值并且被更新的列占用的存储空间在修改前后未发生变化，则先在B+树中定位到这条记录的位置，然后再获取一下记录的X锁，最后在原记录的位置进行修改操作。其实我们也可以把这个定位待修改记录在B+树中位置的过程看成是一个获取X锁的锁定读。
如果未修改该记录的键值并且至少有一个被更新的列占用的存储空间在修改前后发生变化，则先在B+树中定位到这条记录的位置，然后获取一下记录的X锁，将该记录彻底删除掉（就是把记录彻底移入垃圾链表），最后再插入一条新记录。这个定位待修改记录在B+树中位置的过程看成是一个获取X锁的锁定读，新插入的记录由INSERT操作提供的隐式锁进行保护。
如果修改了该记录的键值，则相当于在原记录上做DELETE操作之后再来一次INSERT操作，加锁操作就需要按照DELETE和INSERT的规则进行了。
**INSERT**
 一般情况下，新插入一条记录的操作并不加锁，不过InnoDB通过一种称之为隐式锁的东西来保护这条新插入的记录在本事务提交前不被别的事务访问
### 多粒度锁
前面提到的锁都是针对记录的，也可以被称之为行级锁或者行锁，对一条记录加锁影响的也只是这条记录而已，我们就说这个锁的粒度比较细；其实一个事务也可以在表级别进行加锁，自然就被称之为表级锁或者表锁

但是表锁加上会有性能问题，因此有了意向锁：
* 意向共享锁，英文名：Intention Shared Lock，简称IS锁。当事务准备在某条记录上加S锁时，需要先在表级别加一个IS锁。这样可以避免其他事务在获取同一张表中某些记录的排他锁（X锁），从而提高了并发性能。
* 意向独占锁，英文名：Intention Exclusive Lock，简称IX锁。当事务准备在某条记录上加X锁时，需要先在表级别加一个IX锁。这样可以避免其他事务在获取同一张表中某些记录的共享锁或独占锁，从而提高了并发性能。

假设我们有一个表T，有两行数据A和B。如果事务T1想要获取A和B的共享锁，它需要在表级别加一个IS锁。如果此时另一个事务T2想要获取A或B的独占锁，它必须等待T1释放IS锁后才能获取IX锁。这样可以防止两个事务同时修改同一行数据，从而保证了数据的一致性。

IS、IX锁是表级锁，它们的提出仅仅为了在之后加表级别的S锁和X锁时可以快速判断表中的记录是否被上锁，以避免用遍历的方式来查看表中有没有上锁的记录，也就是说其实IS锁和IX锁是兼容的，IX锁和IX锁是兼容的


### 行锁
#### 记录锁
当一个事务锁定了一条记录时，其他事务不能修改或删除这条记录，直到第一个事务提交或回滚其更改。

记录锁是基于索引的，也就是说，当对某个索引上的数据行进行操作时MySQL会自动获取或释放相应的记录锁
对于使用非唯一索引的数据行，InnoDB引擎使用的是共享锁（Shared Lock），多个事务可以同时对相同的数据行进行读操作。
对于唯一索引的数据行，InnoDB引擎使用的是排他锁（Exclusive Lock），一次只能有一个事务对数据行进行写操作。
#### Gap Locks（间隙锁）
Gap Locks是一种锁定机制，用于防止幻读。当一个事务执行范围查询（例如SELECT … WHERE … BETWEEN … AND … FOR UPDATE或SELECT … WHERE … > … FOR UPDATE）时，InnoDB会对查询结果集中的每个行加上一个gap lock，以防止其他事务在这个范围内插入新的行。
虽然有共享gap锁和独占gap锁这样的说法，但是它们起到的作用都是相同的
#### Next-Key Locks（next-key锁）
Next-Key Locks结合了记录锁（Record Lock）和间隙锁（Gap Locks），用于处理范围查询时的并发问题。
Next-Key Locks的特点和工作原理如下：
* 当一个事务执行范围查询时，InnoDB会获取一个Next Lock来锁住范围内的数据。Next-Key Lock是一个组合锁，将记录锁和间隙锁结合在一起。
* 记录锁（Record Lock）用于锁定查询范围内的已存在的数据行，以确保其他事务无对数据行进行修改或删除。
* 间隙锁（Gap Lock）用于锁定查询范围之间的间隙，以防止其他事务在范围内插入新的数据行。
* Next-Key Lock可以同时锁定一个范围内的数据和间隙，以保证查询结果的一致性和避免幻读（Phantom Read）的问题。
根据范围查询的起始点和结束点，InnoDB会自动获取和释放Next-Key Lock。当事务持有Next-Key Lock时其他事务无法插入或者修改该范围内的数据。
**它既能保护该条记录，又能阻止别的事务将新记录插入被保护记录前面的间隙。**
#### 插入意向锁
插入意向锁并不会阻止别的事务继续获取该记录上任何类型的锁

## InnoDB锁的内存结构
基本结构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b6b8b65f06cb4a6cbd49cb34186dfe1b.png)


* 锁所在的事务信息：不论是表锁还是行锁，都是在事务执行过程中生成的，哪个事务生成了这个锁结构，这里就记载着这个事务的信息。
* 索引信息：对于行锁来说，需要记录一下加锁的记录是属于哪个索引的。
* 锁／行锁信息：
表锁结构和行锁结构在这个位置的内容是不同的：
表锁：
记载着这是对哪个表加的锁，还有其他的一些信息。
行锁：
记载了三个重要的信息：
Space ID：记录所在表空间。
Page Number：记录所在页号。
n_bits：对于行锁来说，一条记录就对应着一个比特位，一个页面中包含很多记录，用不同的比特位来区分到底是哪一条记录加了锁。为此在行锁结构的末尾放置了一堆比特位，这个n_bits属性代表使用了多少比特位。
* type_mode：这是一个32位的数，被分成了lock_mode、lock_type和rec_lock_type三个部分
* 锁的模式（lock_mode），占用低4位，可选的值如下：
LOCK_IS（十进制的0）：表示共享意向锁，也就是IS锁。
LOCK_IX（十进制的1）：表示独占意向锁，也就是IX锁。
LOCK_S（十进制的2）：表示共享锁，也就是S锁。
LOCK_X（十进制的3）：表示独占锁，也就是X锁。
LOCK_AUTO_INC（十进制的4）：表示AUTO-INC锁。
* 锁的类型（lock_type），占用第5～8位，不过现阶段只有第5位和第6位被使用：
LOCK_TABLE（十进制的16），也就是当第5个比特位置为1时，表示表级锁。
LOCK_REC（十进制的32），也就是当第6个比特位置为1时，表示行级锁。
* 行锁的具体类型（rec_lock_type），使用其余的位来表示。只有在lock_type的值为LOCK_REC时，也就是只有在该锁为行级锁时，才会被细分为更多的类型
* LOCK_WAIT（十进制的256） ：也就是当第9个比特位置为1时，表示is_waiting为true，也就是当前事务尚未获取到锁，处在等待状态；当这个比特位为0时，表示is_waiting为false，也就是当前事务获取锁成功。

摘录链接
[Mysql是怎样运行的](https://relph1119.github.io/mysql-learning-notes/#/mysql/26-写作本书时用到的一些重要的参考资料)
