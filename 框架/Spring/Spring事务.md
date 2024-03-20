@[toc]
# Spring事务
## 什么是事务？
事务是逻辑上的一组操作，要么都执行，要么都不执行。  
需要格外注意的是：事务能否生效数据库引擎是否支持事务是关键。比如常用的 MySQL 数据库默认使用支持事务的 innodb引擎。但是，如果把数据库引擎变为 myisam，那么程序也就不再支持事务了！  
我们系统的每个业务方法可能包括了多个原子性的数据库操作，比如下面的 savePerson() 方法中就有两个原子性的数据库操作。这些原子性的数据库操作是有依赖的，它们要么都执行，要不就都不执行。	  
```
public void savePerson() {
		personDao.save(person);
		personDetailDao.save(personDetail);
	}
```
事务最经典也经常被拿出来说例子就是转账了。假如小明要给小红转账 1000 元，这个转账会涉及到两个关键操作就是：将小明的余额减少 1000 元。将小红的余额增加 1000 元。万一在这两个操作之间突然出现错误比如银行系统崩溃或者网络故障，导致小明余额减少而小红的余额没有增加，这样就不对了。事务就是保证这两个关键操作要么都成功，要么都要失败。  
```
public class OrdersService {
	private AccountDao accountDao;

	public void setOrdersDao(AccountDao accountDao) {
		this.accountDao = accountDao;
	}

  @Transactional(propagation = Propagation.REQUIRED,
                isolation = Isolation.DEFAULT, readOnly = false, timeout = -1)
	public void accountMoney() {
    //小红账户多1000
		accountDao.addMoney(1000,xiaohong);
		//模拟突然出现的异常，比如银行中可能为突然停电等等
    //如果没有配置事务管理的话会造成，小红账户多了1000而小明账户没有少钱
		int i = 10 / 0;
		//小王账户少1000
		accountDao.reduceMoney(1000,xiaoming);
	}
}

```
## 事务的特性（ACID）了解么
原子性（Atomicity）：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；一致性（Consistency）：执行事务前后，数据保持一致，例如转账业务中，无论事务是否成功，转账者和收款人的总额应该是不变的；隔离性（Isolation）：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；持久性（Durability）：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。
只有保证了事务的持久性、原子性、隔离性之后，一致性才能得到保障。也就是说 A、I、D 是手段，C 是目的！  
## 详谈 Spring 对事务的支持  
这里再多提一下一个非常重要的知识点：MySQL 怎么保证原子性的？我们知道如果想要保证事务的原子性，就需要在异常发生时，对已经执行的操作进行回滚，在 MySQL 中，恢复机制是通过 回滚日志（undo log） 实现的，所有事务进行的修改都会先记录到这个回滚日志中，然后再执行相关的操作。如果执行过程中遇到异常的话，我们直接利用 回滚日志 中的信息将数据回滚到修改之前的样子即可！并且，回滚日志会先于数据持久化到磁盘上。这样就保证了即使遇到数据库突然宕机等情况，当用户再次启动数据库的时候，数据库还能够通过查询回滚日志来回滚之前未完成的事务。  
### Spring 支持两种方式的事务管理  
#### 编程式事务管理
两种实现  
TransactionManager和TransactionTemplate两种方式  
#### 声明式事务  
XML方式  
java  
#### 混合配置（常用）
xml+注解  

## 三个重要组件
Spring 框架中，事务管理相关最重要的 3 个接口如下：  
PlatformTransactionManager：（平台）事务管理器，Spring 事务策略的核心。  
TransactionDefinition：事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)。  
TransactionStatus：事务运行状态。  
我们可以把 PlatformTransactionManager 接口可以被看作是事务上层的管理者，而 TransactionDefinition 和 TransactionStatus 这两个接口可以看作是事务的描述。PlatformTransactionManager 会根据 TransactionDefinition 的定义比如事务超时时间、隔离级别、传播行为等来进行事务管理 ，而 TransactionStatus 接口则提供了一些方法来获取事务相应的状态比如是否新事务、是否可以回滚等等。  

### PlatformTransactionManager
定义了三个接口
Spring 并不直接管理事务，而是提供了多种事务管理器 。Spring 事务管理器的接口是：PlatformTransactionManager 。通过这个接口，Spring 为各个平台如：JDBC(DataSourceTransactionManager)、Hibernate(HibernateTransactionManager)、JPA(JpaTransactionManager)等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。  
PlatformTransactionManager接口中定义了三个方法：package org.springframework.transaction;  
```
import org.springframework.lang.Nullable;

public interface PlatformTransactionManager {
    //获得事务
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
    //提交事务
    void commit(TransactionStatus var1) throws TransactionException;
    //回滚事务
    void rollback(TransactionStatus var1) throws TransactionException;
}
```
为什么要定义或者说抽象出来PlatformTransactionManager这个接口呢？主要是因为要将事务管理行为抽象出来，然后不同的平台去实现它，这样我们可以保证提供给外部的行为不变，方便我们扩展。  


### Transactionstatus
事务的基本状态  
TransactionStatus接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息。PlatformTransactionManager.getTransaction(…)方法返回一个 TransactionStatus 对象。  
```
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事务
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
}
```

### TransactionDefinition
定义了基本规则  
事务管理器接口 PlatformTransactionManager 通过 getTransaction(TransactionDefinition definition) 方法来得到一个事务，这个方法里面的参数是 TransactionDefinition 类 ，这个类就定义了一些基本的事务属性。什么是事务属性呢？ 事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。  
事务属性包含了 5 个方面：  
隔离级别  
传播行为  
回滚规则  
是否只读  
事务超时  
TransactionDefinition 接口中定义了 5 个方法以及一些表示事务属性的常量比如隔离级别、传播行为等等。
```
package org.springframework.transaction;
import org.springframework.lang.Nullable;
public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    int TIMEOUT_DEFAULT = -1;
    // 返回事务的传播行为，默认值为 REQUIRED。
    int getPropagationBehavior();
    //返回事务的隔离级别，默认值是 DEFAULT
    int getIsolationLevel();
    // 返回事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
    int getTimeout();
    // 返回是否为只读事务，默认值为 false
    boolean isReadOnly();

    @Nullable
    String getName();
}
```
## 隔离性
我们已经知道了一个事务并不是一个单一的操作，它其实包含了多步操作，同时事务的执行的也可以是并发的，就是同时开启多个事务，进行业务操作，并发执行的事务里面的多步操作对着相同的数据进行查找修改，如果没有一个规则，在高并发的环境下可以引发的结果就可想而知，这个时候我们就需要定义一种规则，根据不同的业务场景来保证我们数据的可控性，这种规则就是事务的隔离级别，我们通俗的讲，事务的隔离级别就是一种控制并发执行的事务对数据操作的规则。  
在标准SQL（SQL 92）中定义了四种事务的隔离级别：READ UNCOMMITTED（读未提交）、READ COMMITTED（读已提交）、REPEATABLE READ（可重复读）、SERIALIZABLE（串行化），其中隔离级别最宽松的是读未提交，最严格的是可串行化，当事务隔离级别较低时会引起一些数据问题（后文会讲解），当事务隔离级别设置为可串行化的时候，也就意味着事务的执行就类似于串行了，这个时候性能就会受到影响，所以在实际的业务中，是根据自己的需求来设置合理的事务的隔离级别，在性能和数据安全的两者之间找一个平衡点  
在常用的关系型数据库中Oracle的事务隔离级别就只有三种：读已提交、串行化、只读（Read-Only）。在Oracle中增加了一个只读级别，而去掉了读未提交和可重复读两个级别。  
另外几种常用的关系型数据库MySQL，MariaDB，PostgreSQL都是提供了按照标准SQL的四种隔离级别。  

###  脏读
### 不可重复读
场景如下：  
某电商平台做活动，用户在平台消费5000-10000元的送一个手机，消费超过10000元的送电脑。现在有生成获奖用户报表的事务如下：
1.开始事务  
2.查询消费在5000到10000元的用户  
3.打印送手机用户名单  
4.查询消费在10000元以上的用户  
5.打印送电脑用户名单  
6.结束事务  
锤子是该电商平台的用户，锤子之前的消费是6000元，在上述第2步的时候，锤子符合送手机的条件，而在上述第3步操作的时候，锤子又在电商平台消费了5000元，那么上述事务走到第4步的时候，锤子也符合了送电脑的条件，那么最终锤子就即获得了手机又获得了电脑，这个时候锤子是高兴了，可是电商平台就不高兴了，要吊起来打写这个功能的程序员哥哥喽。  
### 幻读
那么下面咱们就来举栗子说明一下幻读  
场景如下：  
某电商平台还在做活动，用户在平台消费5000-10000元的送一个手机，消费超过10000元的送电脑。现在有生成获奖用户报表的事务如下：  
1.开始事务  
2.查询消费在5000到10000元的用户  
3.打印送手机用户名单  
4.查询消费在10000元以上的用户  
5.打印送电脑用户名单  
6.结束事务  
这个时候锤子和郝大都还不是该电商平台的用户，他们看到好消息，都想参加这个活动，于是两个人都去注册用户并消费。重点来了生成中间获奖名单的事务执行到第3步的时刻，锤子和郝大在此刻都注册了用户，并且锤子消费了6000元，郝大消费了12000元，两个人都以为可以得到自己想要的奖品了，结果最后中奖名单出来后，发现郝大获得了电脑，而锤子什么也没有，可是明明锤子和郝大一起注册的用户，但是郝大却获得了奖品，锤子却没获得。  
上面描述的这种现象就是读已提交隔离级别下的一种幻读现象，两个用户同时注册（同时向表中插入数据），且各自都符合不同的奖品条件要求，但是一个有奖品，一个没有奖品，就会让人感觉，这个福利有内幕的感觉。这就是读已提交下幻读造成的一种影响  

同样上面的场景，如果事务隔离级别提高到可重复读，那么在不改变上述流程的情况下，在MySQL下就不会出现幻读了，因为他们的注册事务是在生成中奖名单之后，所以郝大和锤子都不会有奖品。因为在MySql的设计中可重复读的事务隔离级别的数据读取是快照读，即使其他事务进行insert或是delete，在当前事务中仅仅读取的话是读不到其他事务提交的数据，但是这并不代表MySQL中的可重复读隔离级别就可以完全避免幻读了。  

上面的场景下，我们提升事务隔离级别到可重复读，然后再修改一下生产获奖名单的事务，在第3步的后面添加一步update的操作（将用户表中所有用户记录的更新时间都更新一下），那么在update之后，再执行查询消费在10000元以上的用户的时候，郝大的数据又会被查出来，这个时候，又出现了，同时注册的两个人郝大有奖品，锤子没有奖品。  

那么上面为什么进行一次update后，郝大的数据又会被查出来呢？  

想知道这个原因还要知道两个概念：当前读和快照读（详解如下）  

当前读：读取的是最新版本数据, 会对读取的记录加锁, 阻塞并发事务修改相同记录，避免出现安全问题。  
快照读：可能读取到的数据不是最新版本而是历史版本，而且读取时不会加锁。  
现在知道了这两个概念，那么下面就描述一下，MySQL在可重复读的隔离级别下开启事务时，默认是使用的快照读，所以在整个事务中如果只有查询，那么查询的都是快照数据，就不会受到其他事务影响，但是我们上面又在事务中添加了一个更新语句，当进行更新时快照读就会变成当前读，因为在事务中更新数据是需要对数据进行加锁，直到事务提交才会释放锁，所有由快照读变为当前读后，读取的数据就是最新的，也就把后来添加的郝大账户计算了进去。  

### READ UNCOMMITTED——读未提交  
事务执行过程中可以读取到并发执行的其他事务中未提交的数据。在这种事务隔离级别下可能会引起的问题包括：脏读，不可重复读和幻读。  
未提交，读取到了其他事务没有提交的数据，所以在其他事务回滚的时候，当前事务读取的数据就成了脏数据  
### READ COMMITTED——读已提交  
事务执行过程中只能读到其他事务已经提交的数据。读已提交保证了并发执行的事务不会读到其他事务未提交的修改数据，避免了脏读问题。在这种事务隔离级别下可能引发的问题包括：不可重复读和幻读  
设置场景中事务的隔离级别是读已提交，那么现象是什么样的呢？  
时序描述如下（不可重复读）  
T1 时刻，开启事务1  
T2 时刻，事务1，查询A表id=1的记录的money，结果为money=20  
T3 时刻，开启事务2  
T4 时刻，事务2，更新A表id为1的记录的money为100  
T5 时刻，事务1，查询A表id=1的记录的money，结果为money=20  
T6 时刻，事务2，更新A表id为1的记录的money为200  
T7 时刻，事务1，查询A表id=1的记录的money，结果为money=20  
T8 时刻，事务2提交  
T9 时刻，事务1，查询A表id=1的记录的money，结果为money=200（读取到事务2提交的数据）  
T10 时刻，事务1提交  
在读已提交的事务隔离级别下，事务1不会读到事务2中修改但未提交的数据，在T8时刻事务2提交后，T9时刻事务1读取的数据才发生变化，这种现象就是读已提交。  
### REPEATABLE READ——可重复读
当前事务执行开始后，所读取到的数据都是该事务刚开始时所读取的数据和自己事务内修改的数据。这种事务隔离级别下，无论其他事务对数据怎么修改，在当前事务下读取到的数据都是该事务开始时的数据，所以这种隔离级别下可以避免不可重复读的问题，但还是有可能出现幻读  
幻读就是你一个事务用一样的 SQL 多次查询，结果每次查询都会发现查到一些之前没看到过的数据。注意，幻读特指的是你查询到了之前查询没看到过的数据。此时说明你是幻读了  
设置场景中事务的隔离级别是可重复读，那么现象是什么样的呢？  
时序描述如下  
T1 时刻，开启事务1  
T2 时刻，事务1，查询A表id=1的记录的money，结果为money=20  
T3 时刻，开启事务2  
T4 时刻，事务2，更新A表id为1的记录的money为100  
T5 时刻，事务1，查询A表id=1的记录的money，结果为money=20  
T6 时刻，事务2，更新A表id为1的记录的money为200  
T7 时刻，事务1，查询A表id=1的记录的money，结果为money=20  
T8 时刻，事务2提交  
T9 时刻，事务1，查询A表id=1的记录的money，结果为money=20  
T10 时刻，事务1提交  
从上面描述看出，不论事务2对数据进行了几次更新，到后面即使事务2进行事务提交，事务1里面始终读取的还是自己开始事务前的数据，这种情况就是一种快照读（类似于Oracle下的Read-Only级别），但是这种情况下事务中也是可以进行insert，update和delete操作的。  
### SERIALIZABLE——串行化  
事务的执行是串行执行。这种隔离级别下，可以避免脏读、不可重复读和幻读等问题，但是由于事务一个一个执行，所以性能就比较低。
  
所以在这种隔离级别下，上面的场景中是事务1执行完成后，才会执行事务2，两个事务在执行时不会相互有影响。

### Spring事务隔离级别（五种）
ISOLATION_DEFAULT  
使用数据库的默认级别，对于Oracle来说默认事务隔离级别是读已提交，对于MySQL来说默认事务隔离级别是可重复读，使用方式就是@Transactional(isolation = Isolation.DEFAULT)  
ISOLATION_READ_UNCOMMITTED  
读未提交，含义上文有讲，这里不再赘述，看下源码注释  
ISOLATION_READ_COMMITTED  
读已提交  
ISOLATION_REPEATABLE_READ  
可重复读  
ISOLATION_SERIALIZABLE  
串行化  
## 问题：
可重复读怎么解决不可重读读问题的，原理是什么？  
幻读与不可重复读问题的区别是什么？  
快照读怎么解决幻读的？  


## 事务的传播性
简单的理解就是多个事务方法相互调用时,事务如何在这些方法间传播。  
举个栗子，方法A是一个事务的方法，方法A执行过程中调用了方法B，那么方法B有无事务以及方法B对事务的要求不同都会对方法A的事务具体执行造成影响，同时方法A的事务对方法B的事务执行也有影响，这种影响具体是什么就由两个方法所定义的事务传播类型所决定。
在Spring中对于事务的传播行为定义了七种类型分别是：REQUIRED、SUPPORTS、MANDATORY、REQUIRES_NEW、NOT_SUPPORTED、NEVER、NESTED。  
### REQUIRED
两个方法公用一个事务  
如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务  
### REQUIRED_NEW  
创建一个新事务，如果存在当前事务，则挂起该事务。  
可以理解为设置事务传播类型为REQUIRES_NEW的方法，在执行时，不论当前是否存在事务，总是会新建一个事务。  
场景：  
如果A方法调用B方法
A方法事务传播特性为REQUIRED，B方法传播特性为REQUIRED_NEW  
那么两个数据同时更新一个表的两条数据，这个时候如果，where 后面的条件是索引，那么是行锁，如果不是索引，那么锁定的是表锁。
这个时候这两个方法A方法开启了事务1，然后遇到REQUIRED_NEW开辟事务B，事务A没有将数据提交，将整个表都锁住了，事务B无法获取这个锁，导致一直卡在那。  

还需要注意A方法调用B方法，如果B方法啊抛异常，那么A，B都会回滚，但是如果A抛异常，那么B不会回滚。因为B方法属于A。  

如果事务A要对异常捕获并且处理的话就不会回滚  
```
@Transactional
    public void transfer1() {
        try {
            jdbcTemplate.update("update t_user set user_age = 10 where user_id = ?", 2);
            transferService3.transfer1();
            int i = 1/0;
        }catch (Exception e){
            e.printStackTrace();
        }

    }
```
```
 @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void transfer1() {
        jdbcTemplate.update("update t_user set user_age = 10 where user_id = ?", 1);
//        int i = 1/0;
    }
```
### NESTED  
如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样（开启一个事务）
它实际上是作为一个子事务运行的，调用这个方法的事务是主事务，如果主事务出现问题了，子事务就会回滚，但是子事务出现问题，主事务不会回滚，这个现象正好与REQUIRED_NEW是相反的，但本质是是不一样的。一个是嵌套事务，另一个是新事务。  

### MANDOTARY
如果当前存在事务，则加入改事务，如果不存在，则抛出异常  
适用一些执行该方法必须要有事务的情形  

### NEVER
不使用事务，如果当前事务存在，则抛出异常  
很容易理解，就是我这个方法不使用事务，并且调用我的方法也不允许有事务，如果调用我的方法有事务则我直接抛出异常。  

### SUPPORTED
如果当前存在事务，则加入改事务；如果当前没有事务，则以非事务方式运行  
这个实际相当于合并成了一个事务，任何一个地方抛出异常则全部回滚。  
或者就没有事务。  
### NOT_SUPPORTED
以非事务方式运行，如果当前存在事务，则把当前事务挂起  
也是是说A事务如果抛异常了，不影响B事务的正常运行。  
## 事务的回滚规则
rollbackfor来设置哪种异常回滚规则  
noRollbackFor来设置那种异常不回滚  
默认情况下只有在RuntimeException及其子类或者error时才回滚，可以通过上面的参数来确定回滚规则  

## 事务的只读属性
readOnly属性：设置不能看到其它事务的修改  
可设置true，false  
## 事务的超时时间  
timeout属性  
设置方式  
timeout=3000，单位时秒  
如果不设置，默认的超时时间是-1（不限时间）  

## 注意实现
@Transaction只有在public方法上才能生效  
非事务方法调用事务方法时，事务方法啊的事务不生效  