首先,查看JdbcTempalte源码

```java
public class JdbcTemplate extends JdbcAccessor implements JdbcOperations
```
可以知道实现JdbcOperations的接口,继承JdbcAccessor接口
其中JdbcAccessor接口定义了一些列增删查改方法,在JdbcTemplate实现

```java
public interface JdbcOperations {
    @Nullable
    <T> T execute(ConnectionCallback<T> action) throws DataAccessException;

    @Nullable
    <T> T execute(StatementCallback<T> action) throws DataAccessException;

    void execute(String sql) throws DataAccessException;

    <T> List<T> query(String sql, RowMapper<T> rowMapper) throws DataAccessException;
 
    @Nullable
    <T> T queryForObject(String sql, owMapper<T> rowMapper) throws DataAccessException;
    .......
    JdbcAccessor定义了一些初始化方式,包括对datasource的初始换,是否懒加载(lazyInit)的初始化
```java
 @Nullable
    public DataSource getDataSource() {
        return this.dataSource;
    }
    public void setLazyInit(boolean lazyInit) {
        this.lazyInit = lazyInit;
    }
```
在此基础上,JdbcTemplate形成三种构造方法

```java
    public JdbcTemplate() {
    }

    public JdbcTemplate(DataSource dataSource) {
        this.setDataSource(dataSource);
        this.afterPropertiesSet();
    }

    public JdbcTemplate(DataSource dataSource, boolean lazyInit) {
        this.setDataSource(dataSource);
        this.setLazyInit(lazyInit);
        this.afterPropertiesSet();
    }
```
在这里设置通过调用 this.setDataSource(dataSource);进行设置数据源,但是需要注意此时并没有建立连接，JdbcTemplate在实现JdbcAccessor接口的基础上进行建立连接，并执行sql语句的，这里以execute方法为例介绍，源代码如下

```java
 @Nullable
    private <T> T execute(StatementCallback<T> action, boolean closeResources) throws DataAccessException {
        Assert.notNull(action, "Callback object must not be null");
        Connection con = DataSourceUtils.getConnection(this.obtainDataSource()); //这句建立连接
        Statement stmt = null;

        Object var12;
        try {
            stmt = con.createStatement();
            this.applyStatementSettings(stmt);
            T result = action.doInStatement(stmt);
            this.handleWarnings(stmt);
            var12 = result;
        } catch (SQLException var10) {
            String sql = getSql(action);
            JdbcUtils.closeStatement(stmt);
            stmt = null;
            DataSourceUtils.releaseConnection(con, this.getDataSource());
            con = null;
            throw this.translateException("StatementCallback", sql, var10);
        } finally {
            if (closeResources) {
                JdbcUtils.closeStatement(stmt);
                DataSourceUtils.releaseConnection(con, this.getDataSource());
            }

        }

        return var12;
    }

```
在 Connection con = DataSourceUtils.getConnection(this.obtainDataSource());中建立其与数据库的连接，具体实现为：

```java
public abstract class DataSourceUtils {
    public static final int CONNECTION_SYNCHRONIZATION_ORDER = 1000;
    private static final Log logger = LogFactory.getLog(DataSourceUtils.class);

    public DataSourceUtils() {
    }

    public static Connection getConnection(DataSource dataSource) throws CannotGetJdbcConnectionException {
        try {
            return doGetConnection(dataSource);
        } catch (SQLException var2) {
            throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection", var2);
        } catch (IllegalStateException var3) {
            throw new CannotGetJdbcConnectionException("Failed to obtain JDBC Connection: " + var3.getMessage());
        }
    }
    。。。。。。。。
}
```
通过建立一个静态方法实现了连接，并执行语句。

问题：在JdbcTemplate中通过构造DataSourceUtils()来建立连接的，然而在执行完成sql语句后会进行关闭

```java
 JdbcUtils.closeStatement(stmt);
 DataSourceUtils.releaseConnection(con, this.getDataSource());
```
在源码中没有看出是否进行close方法的关闭，还是暂时挂起式关闭，如果是前者的话每次执行一个sql语句就会产生大量时间消耗，通过线程池技术可以解决这个问题，那该如何耦合呢？