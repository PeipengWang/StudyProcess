# Jdbc增删查改
所需jar包
```
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.25</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```
增删查改代码
```
import org.junit.Test;

import java.sql.*;

public class JdbcTest {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        //1、注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //2、创建连接
        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1/jdbc_demo","root","123456");
        //3、sql语句预处理
        String sql = "insert into t_user(user_name,user_age) values (?,?)";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //4、给占位符赋值
        preparedStatement.setString(1,"张三");
        preparedStatement.setInt(2,18);
        //5、执行更新操作
        preparedStatement.executeUpdate();
        //6、关闭
        preparedStatement.close();
        connection.close();

    }
    @Test
    public void addTest() throws ClassNotFoundException, SQLException {
        //1、注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //2、创建连接
        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1/jdbc_demo","root","123456");
        //3、sql语句预处理
        String sql = "insert into t_user(user_name,user_age) values (?,?)";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //4、给占位符赋值
        preparedStatement.setString(1,"张三");
        preparedStatement.setInt(2,18);
        //5、执行更新操作
        preparedStatement.executeUpdate();
        //6、关闭
        preparedStatement.close();
        connection.close();
    }
    @Test
    public void selectTest() throws ClassNotFoundException, SQLException {
        //1、注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //2、创建连接
        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1/jdbc_demo","root","123456");
        //3、sql语句预处理
        String sql = "select * from t_user where user_id=?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //4、给占位符赋值
        preparedStatement.setString(1,"1");
        //5、执行查询操作操作放到resultSet中
        ResultSet resultSet = preparedStatement.executeQuery();
        //6、遍历查询操作结果
        while (resultSet.next()){
            System.out.println(resultSet.getInt(1));
            System.out.println(resultSet.getString(2));
            System.out.println(resultSet.getInt(3));
        }
        //7、关闭
        preparedStatement.close();
        connection.close();
    }
    @Test
    public void updateTest() throws ClassNotFoundException, SQLException {
        //1、注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //2、创建连接
        Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1/jdbc_demo","root","123456");
        //3、sql语句预处理
        String sql = "update t_user set user_name = ? , user_age = ? where user_id = ?";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //4、给占位符赋值
        preparedStatement.setString(1,"王四");
        preparedStatement.setInt(2,25);
        preparedStatement.setInt(3,1);
        //5、执行更新操作
        preparedStatement.executeUpdate();
        //6、关闭
        preparedStatement.close();
        connection.close();
    }
}

```
问题:每次都重复新建连接，关闭
解决：提取工具类到jdbcUtil
通过提取连接的方式对这个连接进行重复使用，从而消除重复获取连接进行的大量时间损耗