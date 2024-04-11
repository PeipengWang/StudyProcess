
# Spring事务管理
## 是什么
事务是逻辑上的一组操作，要么都执行，要么都不执行。
另外，需要格外注意的是：事务能否生效数据库引擎是否支持事务是关键。比如常用的 MySQL 数据库默认使用支持事务的 innodb引擎。但是，如果把数据库引擎变为 myisam，那么程序也就不再支持事务了！
## 实例
事务最经典也经常被拿出来说例子就是转账了。
假如小明要给小红转账 1000 元，这个转账会涉及到两个关键操作就是：


将小明的余额减少 1000 元。
将小红的余额增加 1000 元。


万一在这两个操作之间突然出现错误比如银行系统崩溃或者网络故障，导致小明余额减少而小红的余额没有增加，这样就不对了。事务就是保证这两个关键操作要么都成功，要么都要失败。

## ACID
事务的特性（ACID）了解么?

原子性（Atomicity）： 一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
一致性（Consistency）： 在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设约束、触发器、级联回滚等。
隔离性（Isolation）： 数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read uncommitted）、提交读（read committed）、可重复读（repeatable read）和串行化（Serializable）。
持久性（Durability）: 事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。
## Spring的两种事务管理方法
方法一：编程式事务管理
通过 TransactionTemplate或者TransactionManager手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。
方法二：声明式事务管理
推荐使用（代码侵入性最小），实际是通过 AOP 实现（基于@Transactional 的全注解方式使用最多）。

使用 @Transactional注解进行事务管理的示例代码如下：
首先查看Transactional注解定义
```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional 
```
## 什么是事务传播行为？
那调用链中的子方法，是用一个新事务，还是使用当前事务呢？这个子方法决定使用新事务还是当前事务（或不使用事务）的策略，就叫事务传播。
在 Spring 的事务管理中，这个子方法的事务处理策略叫做事务传播行为（Propogation Behavior）。
但给这些传播行为分类之后，无非是以下三种：

优先使用当前事务
不使用当前事务，新建事务
不使用任何事务
例如：
以 Spring JDBC + Spring 注解版的事务举例。在默认的事务传播行为下，methodA 和 methodB 会使用同一个 Connection，在一个事务中

```
@Transactional
public void methodA(){
    jdbcTemplate.batchUpdate(updateSql, params);
    methodB();
}

@Transactional
public void methodB(){
    jdbcTemplate.batchUpdate(updateSql, params);
}
```
如果我想让 methodB 不使用 methodA 的事务，自己新建一个连接/事务呢？只需要简单的配置一下 @Transactional 注解：

```
@Transactional
public void methodA(){
    jdbcTemplate.batchUpdate(updateSql, params);
    methodB();
}

// 传播行为配置为 - 方式2，不使用当前事务，独立一个新事务
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB(){
    jdbcTemplate.batchUpdate(updateSql, params);
}
```
就是这么简单，获取 Connection/多方法共享 Connection/多方法共享+独享 Connection/提交/释放连接之类的操作，完全不需要我们操心，Spring 都替我们做好了。
## 怎么回滚？
在注解版的事务管理中，默认的的回滚策略是：抛出异常就回滚。这个默认策略挺好，连回滚都帮我们解决了，再也不用手动回滚。
但是如果在嵌套事务中，子方法独立新事务呢？这个时候哪怕抛出异常，也只能回滚子事务，不能直接影响前一个事务
可如果这个抛出的异常不是 sql 导致的，比如校验不通过或者其他的异常，此时应该将当前的事务回滚吗？
这个还真不一定，谁说抛异常就要回滚，异常也不回滚行不行？
当然可以！抛异常和回滚事务本来就是两个问题，可以连在一起，也可以分开处理
```
// 传播行为配置为 - 方式2，不使用当前事务，独立一个新事务

// 指定 Exception 也不会滚
@Transactional(propagation = Propagation.REQUIRES_NEW, noRollbackFor = Exception.class)
public void methodB(){
    jdbcTemplate.batchUpdate(updateSql, params);
}
```
每个事务/连接使用不同配置
除了传播和回滚之外，还可以给每个事务/连接使用不同的配置，比如不同的隔离级别：

```
@Transactional
public void methodA(){
    jdbcTemplate.batchUpdate(updateSql, params);
    methodB();
}

// 传播行为配置为 - 方式2，不使用当前事务，独立一个新事务
// 这个事务/连接中使用 RC 隔离级别，而不是默认的 RR
@Transactional(propagation = Propagation.REQUIRES_NEW, isolation = Isolation.READ_UNCOMMITTED)
public void methodB(){
    jdbcTemplate.batchUpdate(updateSql, params);
}

```
除了隔离级别之外，其他的 JDBC Connection 配置当然也是支持的，比如 readOnly。这样一来，虽然我们不用显示的获取 connection/session，但还是可以给嵌套中的每一个事务配置不同的参数，非常灵活。

## 事务管理器（TransactionManager）模型
事务管理的核心操作只有两个：提交和回滚。前面所谓的传播、嵌套、回滚之类的，都是基于这两个操作。
所以 Spring 将事务管理的核心功能抽象为一个事务管理器（Transaction Manager），基于这个事务管理器核心，可以实现多种事务管理的方式。
这个核心的事务管理器只有三个功能接口：
获取事务资源，资源可以是任意的，比如jdbc connection/hibernate mybatis session之类，然后绑定并存储
提交事务 - 提交指定的事务资源
回滚事务 - 回滚指定的事务资源
```
interface PlatformTransactionManager{
    // 获取事务资源，资源可以是任意的，比如jdbc connection/hibernate mybatis session之类
    TransactionStatus getTransaction(TransactionDefinition definition)
            throws TransactionException;
    // 提交事务
    void commit(TransactionStatus status) throws TransactionException;
    // 回滚事务
    void rollback(TransactionStatus status) throws TransactionException;
}
```
## 事务失效场景

![在这里插入图片描述](https://img-blog.csdnimg.cn/348be7578ac64474907d5157ea145a8e.png)

