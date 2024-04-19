# Mybatis插件
## 四大接口介绍
Mybatis中的插件又叫做拦截器，通过插件可以在Mybatis某个行为执行时进行拦截并改变这个行为。通常，Mybatis的插件可以作用于Mybatis中的四大接口，分别为Executor，ParameterHandler，ResultSetHandler和StatementHandler，归纳如下表所示。

Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed) 拦截执行器的方法  
StatementHandler (prepare, parameterize, batch, update, query) 拦截Sql语法构建的处理  
ParameterHandler (getParameterObject, setParameters) 拦截参数的处理  
ResultSetHandler (handleResultSets, handleOutputParameters) 拦截结果集的处理  

|     可作用接口     |                                            可作用方法                                             |     拦截器用途      |
| ---------------- | ------------------------------------------------------------------------------------------------ | ------------------ |
| Executor         | update()，query()，flushStatements()，commit()，rollback()，getTransaction()，close()，isClosed() | 拦截执行器中的方法   |
| ParameterHandler | getParameterObject()，setParameters()                                                            | 拦截对参数的处理     |
| ResultSetHandler | handleResultSets()，handleOutputParameters()                                                     | 拦截对结果集的处理   |
| StatementHandler | prepare()，parameterize()，batch()，update()，query()                                             | 拦截 SQL 构建的处理 |

## 所需要的注解

@Intercepts 注解是 MyBatis 中用于声明一个类为拦截器的注解。它通常与 @Signature 注解一起使用，用于指定要拦截的方法的签名信息。

@Signature 注解指定了要拦截 StatementHandler 接口的 prepare 方法。会在设置的method方法执行之前先执行intercept()方法
type 属性表示被拦截方法所在的接口或类，这里是 StatementHandler.class。

method 属性表示被拦截的方法名，这里是 prepare。

args 属性表示被拦截方法的参数类型，这里是 {Connection.class, Integer.class}，表示 prepare 方法接受一个 Connection 和一个 Integer 类型的参数。

## Interceptor接口介绍
Interceptor 接口是 MyBatis 提供的拦截器接口，用于拦截 MyBatis 执行器中的方法调用。该接口定义了三个方法

1、Object intercept(Invocation invocation) throws Throwable：

这是拦截器的核心方法，用于实现拦截逻辑。  
invocation 参数是被拦截方法的详细信息，包括目标对象、方法、方法参数等。  
在这个方法中，你可以对拦截的方法进行前置处理、后置处理，或者替换原始的方法执行逻辑。  
你可以通过 invocation.proceed() 方法调用原始的方法，获取原始方法的返回值，并在必要时进行修改。  
2、Object plugin(Object target)： 

用于返回目标对象的代理对象。  
target 参数是被拦截的目标对象，通过该方法返回的代理对象来替代原始的目标对象。  
通常，在该方法中使用 Plugin.wrap(target, this) 来生成代理对象，其中 this 表示当前拦截器实例。  
这个方法的返回值会被传递给 MyBatis，用于替换原始的目标对象。  
3、void setProperties(Properties properties)：  
用于初始化拦截器实例时设置属性。  
properties 参数是从配置文件中读取的属性，可以在这个方法中获取和处理配置信息。  
通常，这个方法被用来配置和初始化拦截器的一些参数。  

## Executor接口
### Executor接口介绍
Executor 接口是执行器的核心接口，它定义了执行 SQL 语句的一系列方法。Executor 接口的主要职责是执行 SQL 语句，并返回执行结果。  

### Executor 接口的一些关键方法

1、int update(MappedStatement ms, Object parameter)：

用于执行更新（INSERT、UPDATE、DELETE）操作的方法。  
ms 参数是包含 SQL 语句信息的 MappedStatement 对象。  
parameter 参数是传递给 SQL 语句的参数对象。  

2、<E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)：  
用于执行查询操作的方法。  
ms 参数是包含 SQL 语句信息的 MappedStatement 对象。  
parameter 参数是传递给 SQL 语句的参数对象。  
rowBounds 参数用于分页查询，指定查询的结果范围。  
resultHandler 参数用于处理查询结果的回调处理器。  
key 参数是缓存键，用于缓存查询结果。  
boundSql 参数是 SQL 语句的封装对象。  

<E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler)：
与上述方法相似，但省略了 CacheKey 和 BoundSql 参数。  
3、void commit(boolean required)：  

提交事务的方法，required 参数表示是否要求事务提交。  

4、void rollback(boolean required)：  
回滚事务的方法，required 参数表示是否要求事务回滚。  

5、CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql)：  
创建用于缓存查询结果的缓存键。  

6、boolean isCached(MappedStatement ms, CacheKey key)：  
检查查询结果是否已经缓存。  

7、void clearLocalCache()：  
清空本地缓存。  

8、void deferLoad(MappedStatement ms, MetaObject resultObject, String property, CacheKey key, Class<?> targetType)：
延迟加载一对一或一对多关联查询的方法。  

### Executor 接口的实现类有多种

包括：
SimpleExecutor：简单的执行器，每执行一次 update 或 select 操作，就开启一个新的 Statement 对象，用完立刻关闭 Statement 对象。

ReuseExecutor：可重用的执行器，执行 update 操作时，会重用 PreparedStatement 对象。

BatchExecutor：批处理执行器，可以进行批处理操作。

MyBatis 默认使用 SimpleExecutor，但你可以在配置文件中进行配置，选择不同的执行器类型。不同的执行器对事务、缓存等的处理方式有所不同，你可以根据项目的需要进行选择。  

### 作为拦截器接口的作用  

拦截器对于 Executor 接口的方法调用提供了一些扩展点，允许你在 SQL 执行的前、后、异常处理等阶段插入自定义逻辑。

1、修改执行的 SQL 语句：  
拦截器可以在执行前或执行后修改 SQL 语句，包括但不限于添加、修改、删除 SQL 语句的某些部分。

2、参数处理：

在执行 SQL 语句前，拦截器可以对用户传递的参数进行修改或校验，以满足业务需求。

3、结果处理：

在执行 SQL 语句后，拦截器可以对查询结果进行处理，例如结果的加工、加密、解密等。

4、异常处理：

拦截器可以捕获 SQL 执行过程中的异常，并进行相应的处理。这包括记录异常日志、抛出特定异常等。

5、延迟加载处理：

拦截器可以在查询时进行延迟加载处理，提前加载相关联的数据，或者在需要时再进行加载。

6、事务处理：

拦截器可以在事务提交或回滚前后执行一些额外的逻辑，例如清理资源、记录事务日志等。

7、日志记录：  
拦截器可以用于记录 SQL 执行过程的日志，用于调试和性能监控。  

使用实例  

```
package com.Intercepts;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.plugin.Intercepts;
import org.apache.ibatis.plugin.Invocation;
import org.apache.ibatis.plugin.Signature;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;

import java.lang.reflect.Method;
import java.util.Properties;

@Intercepts({@Signature(type = Executor.class, method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})})
public class TestInterceptorExector implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("测试Executor，在query之前执行");
        // 获取被拦截的对象
        Object target = invocation.getTarget();
        // 获取被拦截的方法
        Method method = invocation.getMethod();
        // 获取被拦截的方法的参数
        Object[] args = invocation.getArgs();

        // 执行被拦截的方法前，做一些事情

        // 执行被拦截的方法
        Object result = invocation.proceed();

        // 执行被拦截的方法后，做一些事情

        // 返回执行结果
        return result;
    }

    @Override
    public Object plugin(Object target) {
        return Interceptor.super.plugin(target);
    }

    @Override
    public void setProperties(Properties properties) {
        Interceptor.super.setProperties(properties);
    }

}
```
上述定义的接口可以看到：
设置的拦截器为 Executor，会在query之前进行拦截，query方法中有传入参数是args的内容
输入内容
```
09:32:01.292 [main] DEBUG org.apache.ibatis.logging.LogFactory - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
09:32:01.341 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
09:32:01.341 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
09:32:01.341 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
09:32:01.341 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
测试Executor，在query之前执行
09:32:04.916 [main] DEBUG org.apache.ibatis.transaction.jdbc.JdbcTransaction - Opening JDBC Connection
09:32:05.273 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - Created connection 1403700359.
09:32:05.274 [main] DEBUG org.apache.ibatis.transaction.jdbc.JdbcTransaction - Setting autocommit to false on JDBC Connection [org.postgresql.jdbc.PgConnection@53aac487]
09:36:28.605 [main] DEBUG com.dao.UserDao.getUserById - ==> Parameters: 1(Integer)
09:36:28.652 [main] DEBUG com.dao.UserDao.getUserById - <==      Total: 1
People{id=1, name='John Doe', age='25'}
```
可知：在获取JDBC的连接之前就执行了前置方法

## ParameterHandler

### 介绍
ParameterHandler 在 MyBatis 中是用于处理 SQL 语句中参数的接口。它主要负责将用户传递的参数值设置到预编译的 SQL 语句中，以便执行数据库操作。

### 基本方法

ParameterHandler 接口定义了两个主要方法：

1、getParameterObject()：

该方法用于获取用户传递给 SQL 语句的参数对象。这个参数对象可以是一个简单类型，也可以是一个复杂的 Java 对象，取决于 SQL 语句中的参数配置。

2、setParameters(PreparedStatement ps)：

该方法将 getParameterObject() 返回的参数对象中的值设置到预编译的 SQL 语句中。

这包括将参数值设置到占位符（?）中，以确保 SQL 语句在执行时能够正确地与传递的参数值匹配。

### 使用场景
1、修改参数值：
在 ParameterHandler 的 setParameters 方法中，拦截器可以获取到用户传递的参数对象，并对参数值进行修改。这样可以在执行 SQL 语句之前对参数进行调整或加工。  
2、参数校验：  
拦截器可以在 setParameters 方法中对参数进行校验，确保参数的合法性。如果参数不符合预期，拦截器可以抛出异常或进行相应的处理。  
3、参数加密或解密：  
在 setParameters 方法中，拦截器可以对参数进行加密或解密操作，以增强对敏感信息的处理。  
4、参数日志记录：  
拦截器可以在 setParameters 方法中记录参数的日志，用于调试或审计目的。  
4、动态参数生成：  
拦截器可以在 setParameters 方法中根据一些条件动态生成参数，实现更灵活的 SQL 语句构建。  
这个主要是进行参数改写，有个实例可以参考  
[MyBatis拦截器ParameterHandler参数改写-修改时间和修改人统一插入](https://zhuanlan.zhihu.com/p/378558071)
[mybatis运行时拦截ParameterHandler注入参数](https://blog.csdn.net/yangbo787827967/article/details/81562476)

这里只罗列大体框架的写法
```
package com.Intercepts;

import org.apache.ibatis.executor.parameter.ParameterHandler;

import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.plugin.Intercepts;
import org.apache.ibatis.plugin.Invocation;
import org.apache.ibatis.plugin.Signature;

import java.lang.reflect.Method;
import java.sql.PreparedStatement;

@Intercepts({@Signature(type = ParameterHandler.class, method = "setParameters",
        args = {PreparedStatement.class})})
public class TestInterceptorParameterHandler implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("执行ParameterHandler的setParameters的前置方法---");
        // 获取被拦截的对象
        Object target = invocation.getTarget();
        // 获取被拦截的方法
        Method method = invocation.getMethod();
        // 获取被拦截的方法的参数
        Object[] args = invocation.getArgs();
        // 执行被拦截的方法前，做一些事情

        // 执行被拦截的方法
        Object result = invocation.proceed();

        // 执行被拦截的方法后，做一些事情

        // 返回执行结果
        return result;
    }

}
```
## ResultSetHandler

### 简介

在MyBatis中，ResultSetHandler负责以下任务：  

1、从ResultSet中读取数据：ResultSet是数据库查询的结果集，ResultSetHandler通过遍历结果集并读取其中的数据。  

2、数据映射：ResultSetHandler根据配置文件中的映射规则，将ResultSet中的数据映射到Java对象的属性中。这种映射可以是一对一（One-to-One）、一对多（One-to-Many）等关系。

3、构建结果对象：ResultSetHandler根据映射规则，将ResultSet中的数据构建成最终的结果对象，可以是一个Java对象或一个集合。
简而言之，ResultHandler接口处理Statement执行后产生的结果集，生成结果列表，并且处理存储过程执行后的输出参数

不同类型的ResultSetHandler实现类可以处理不同的结果集类型，例如：

BeanHandler：将结果集映射到一个Java对象。  
BeanListHandler：将结果集映射到一个Java对象的集合。  
MapHandler：将结果集映射到一个Map对象。  
MapListHandler：将结果集映射到一个Map对象的集合。  
ScalarHandler：将结果集的单个值映射到一个对象。  
自定义的ResultSetHandler：可以根据具体需求自定义结果集处理方式。  
使用MyBatis时，可以通过配置文件或注解来指定使用哪种ResultSetHandler来处理查询结果。  

ResultSetHandlerInterceptor接口并注册为拦截器，可以在MyBatis执行查询操作时拦截ResultSetHandler的处理过程，并对结果集进行自定义的增强或修改。  
### 方法
ResultSetHandler接口定义了三个方法，它们分别是：

1、handleResultSets(Statement stmt)：  
这个方法用于处理查询操作返回的结果集。它接收一个Statement对象作为参数，表示当前查询操作的语句。
在这个方法中，你可以通过Statement对象获取查询结果集（ResultSet），并对结果集进行处理，例如将结果集映射到Java对象或集合中。

2、handleOutputParameters(CallableStatement cs)：  
这个方法用于处理存储过程调用的输出参数。它接收一个CallableStatement对象作为参数，表示当前存储过程的调用语句。
在这个方法中，你可以通过CallableStatement对象获取存储过程执行后的输出参数，并对其进行处理。

3、handleCursorResultSets(Statement stmt)：  
这个方法用于处理返回的游标（Cursor）结果集。它接收一个Statement对象作为参数，表示当前查询操作的语句。
在这个方法中，你可以通过Statement对象获取游标结果集，并对其进行处理，例如将游标结果集映射到Java对象或集合中。

### 基本使用注解

```
@Intercepts({@Signature(type = ResultSetHandler.class, method = "handleResultSets",
        args = {Statement.class})})
public class TestInterceptorResultSetHandler implements Interceptor 
```

## StatementHandler  
StatementHandler接口定义了一系列方法，用于处理SQL语句的预处理、参数设置、语句执行和结果集处理等操作。  
StatementHandler接口的主要作用是将Java对象与数据库之间进行数据交互，它封装了对JDBC Statement对象的操作，并提供了一些便捷的方法来处理SQL语句。  
在MyBatis中，StatementHandler接口的实现类负责以下任务：  
SQL语句的预处理：根据传递的SQL语句，StatementHandler负责对SQL语句进行预处理，例如参数的替换或占位符的填充。  

参数设置：StatementHandler接口提供了设置SQL语句参数的方法，通过这些方法可以将Java对象中的数据设置到SQL语句中。  

语句执行：StatementHandler负责将预处理后的SQL语句发送给数据库执行，并获取执行结果。

结果集处理：StatementHandler接口提供了处理查询结果集的方法，可以将结果集映射为Java对象或集合。

在MyBatis的配置文件中，可以通过指定不同的StatementHandler实现类来定制SQL语句的执行行为，例如使

用PreparedStatementHandler来处理带有占位符的SQL语句，或者使用CallableStatementHandler来处理存储过程的调用。


一个比较好的调用 StatementHandler进行分页查询的实例
https://www.sundayfine.com/mybatis-pagination/