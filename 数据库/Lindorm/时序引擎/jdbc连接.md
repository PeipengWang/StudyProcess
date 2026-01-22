## 前置条件

- 已安装Java环境，要求安装JDK 1.8及以上版本。
- 已将客户端IP地址添加至Lindorm白名单（window机器有可能自动修改注意添加），具体操作请参见[设置白名单](https://help.aliyun.com/zh/lindorm/getting-started/configure-a-whitelist#task-2045184)。
- 已获取云原生多模数据库 Lindorm时序引擎的连接地址，具体操作请参见[查看连接地址](https://help.aliyun.com/zh/lindorm/user-guide/view-endpoints#task-2143195)。

## 操作步骤

1. 通过以下两种方式安装Lindorm JDBC Driver依赖。

   - 手动安装。

     在本地自行下载JAR包集成JDBC Driver，下载链接为：[Lindorm-all-client](https://repo1.maven.org/maven2/com/aliyun/lindorm/lindorm-all-client/)。选择需要安装的版本，以2.1.5为例，下载lindorm-all-client-2.1.5.jar包。

   - 通过Maven下载。

     如果在Maven项目中集成JDBC Driver，创建Project并在pom.xml中添加以下依赖配置，具体内容如下：

      

     ```xml
     <dependency>
         <groupId>com.aliyun.lindorm</groupId>  
         <artifactId>lindorm-all-client</artifactId>
         <version>2.2.1.3</version>
     </dependency>
     ```

     **说明**

     lindorm-all-client的版本号根据需求填写。

2. 访问Lindorm时序引擎。完整的代码示例如下。

    

   ```java
   import java.sql.*;
   
   class Test {
       public static void main(String[] args) {
           // 此处填写Lindorm时序引擎JDBC连接地址
           String url = "jdbc:lindorm:tsdb:url=http://ld-bp12pt80qr38p****-proxy-tsdb-pub.lindorm.rds.aliyuncs.com:8242"; //公网地址
           Connection conn = null;
           Properties props = new Properties();
           props.setProperty("user", "user");
           props.setProperty("password", "test");
           props.setProperty("database", "hlw_tm");
           try {
               conn = DriverManager.getConnection(url,props);
               try (Statement stmt = conn.createStatement()) {
                   //创建时序数据表，默认访问 default database
                   stmt.execute("CREATE TABLE sensor1 (device_id VARCHAR TAG,region VARCHAR TAG,time TIMESTAMP,temperature DOUBLE,humidity DOUBLE,PRIMARY KEY(device_id))");
   
                   //批量写入数据
                   stmt.addBatch("INSERT INTO sensor1(device_id, region, time, temperature, humidity) values('F07A1260','north-cn','2021-04-22 15:33:00',12.1,45)");
                   stmt.addBatch("INSERT INTO sensor1(device_id, region, time, temperature, humidity) values('F07A1260','north-cn','2021-04-22 15:33:10',13.2,47)");
                   stmt.addBatch("INSERT INTO sensor1(device_id, region, time, temperature, humidity) values('F07A1260','north-cn','2021-04-22 15:33:20',10.6,46)");
                   stmt.addBatch("INSERT INTO sensor1(device_id, region, time, temperature, humidity) values('F07A1261','south-cn','2021-04-22 15:33:00',18.1,44)");
                   stmt.addBatch("INSERT INTO sensor1(device_id, region, time, temperature, humidity) values('F07A1261','south-cn','2021-04-22 15:33:10',19.7,44)");
                   stmt.executeBatch();
                   stmt.clearBatch();
               }
   
               // 使用绑定参数的方式查询数据
               // 强烈建议指定时间范围减少数据扫描
               try (PreparedStatement pstmt = conn.prepareStatement("SELECT device_id, region,time,temperature,humidity FROM sensor1 WHERE time >= ? and time <= ?")) {
                   Timestamp startTime =Timestamp.valueOf("2021-04-22 15:33:00");
                   Timestamp endTime = Timestamp.valueOf("2021-04-22 15:33:20");
                   pstmt.setTimestamp(1, startTime);
                   pstmt.setTimestamp(2, endTime);
                   try (ResultSet rs = pstmt.executeQuery()) {
                       while (rs.next()) {
                           String device_id = rs.getString("device_id");
                           String region = rs.getString("region");
                           Timestamp time = rs.getTimestamp("time");
                           Double temperature = rs.getDouble("temperature");
                           Double humidity = rs.getDouble("humidity");
                           System.out.printf("%s %s %s %f %f\n", device_id, region, time, temperature, humidity);
                       }
                   }
               }
           } catch (SQLException e) {
               // 异常处理需要结合实际业务逻辑编写
               e.printStackTrace();
           } finally {
               try {
                   if (conn != null) {
                       conn.close();
                   }
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   ```

   **说明**

   - JDBC连接地址相关参数说明，请参见[JDBC连接地址说明](https://help.aliyun.com/zh/lindorm/user-guide/jdbc-connection-address-description)。
   - JDBC Driver访问时序引擎时支持的API接口和对应的方法请参见[支持的API接口和方法](https://help.aliyun.com/zh/lindorm/user-guide/use-jdbc-driver-to-connect-to-and-use-lindormtsdb/#section-r8j-wgw-dv6)。
   - Lindorm时序引擎支持的SQL语法请参见[SQL语法参考](https://help.aliyun.com/document_detail/216790.html)。

   执行成功预计返回以下结果：

    

   ```java
   F07A1261 south-cn 2021-04-22 15:33:00.0 18.100000 44.000000
   F07A1261 south-cn 2021-04-22 15:33:10.0 19.700000 44.000000
   F07A1260 north-cn 2021-04-22 15:33:00.0 12.100000 45.000000
   F07A1260 north-cn 2021-04-22 15:33:10.0 13.200000 47.000000
   F07A1260 north-cn 2021-04-22 15:33:20.0 10.600000 46.000000
   ```