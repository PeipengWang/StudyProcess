# jdbcUtil
```
import java.io.FileInputStream;
import java.io.IOException;
import java.sql.*;
import java.util.Properties;

//封装jdbc连接公共代码与资源关闭的公共给代码
public class jdbcUtil {
    //加载配置文件
    private static Properties properties = new Properties();
    //定义连接属性
    private static Connection connection;
    static {
        try {
            //获取配置文件
            properties.load(new FileInputStream("E:\\code\\HandleWrite_connectionPool\\demoConnectionPool\\HandleWrite_jdbcDataSource\\src\\main\\java\\db.properties"));
            String driverClass = properties.getProperty("jdbc.driverClass");
            String url = properties.getProperty("jdbc.url");
            String user = properties.getProperty("jdbc.user");
            String password = properties.getProperty("jdbc.password");
            //加载驱动
            Class.forName(driverClass);
            //创建连接
            connection = DriverManager.getConnection(url,user,password);
        } catch (IOException | ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
    }
    //获得连接方法
    public Connection getConnection(){
        return connection;
    }
    public static void close(Connection connection, PreparedStatement preparedStatement, ResultSet resultSet) throws SQLException {
        if (connection != null){
            connection.close();
        }
        if(preparedStatement != null){
            preparedStatement.close();
        }
        if(resultSet != null){
            resultSet.close();
        }
    }
}

```
查询语句便简化为
```


import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class jdbcUtilTest {
    public static void main(String[] args) throws SQLException {
        long before_time = System.currentTimeMillis();
        long n = 3000;
        jdbcUtil jdbcUtil = new jdbcUtil();
        Connection connection = jdbcUtil.getConnection();
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        for(int i = 0; i < n; i ++) {
            String sql = "select * from t_user where user_id=?";
           preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setInt(1, i);
           resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
               
            }
        }
        jdbcUtil.close(connection, preparedStatement, resultSet);
        long after_time = System.currentTimeMillis();
        System.out.println("执行"+n+"次花费时间:"+ (after_time-before_time));
    }
}


```
通过构造一个jdbcUtil的静态工具类，在这个类中有一个连接，使用这个连接执行  
执行3000次花费时间:1587  
这时发现执行3000此仅仅用了1.5m，一半时间用来获取连接，即实现jdbcUtil的静态类，一半时间来执行sql语句。  
从上面可以有一个思路，尽量减少连接的创建，尤其是在执行业务的时候。然而，在多线程条件下，单纯靠一个静态类，每个线程都要用这个静态类的话，这个类是需要加锁的， 这时候对于这个静态类存在激烈的竞争，大量的时间花在了竞争这个静态类的时间上，这个多线程竞争优化的另一个话题，在这里不再展开。资源太少导致这个静态类成了“香饽饽”，典型的卖方市场，导致通货紧缩，那我们可以多生产几个这样的线程，减少竞争，多建的几个线程集合有个名字叫做“线程池”。  
java对于我们的线程池有一定的规范，方便统一管理。  
