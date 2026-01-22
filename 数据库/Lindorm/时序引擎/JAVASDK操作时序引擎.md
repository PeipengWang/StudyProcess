# 教程：通过Java Native SDK连接并使用Lindorm时序引擎

本文介绍通过Java Native SDK连接并使用Lindorm时序引擎的方法。

## 前提条件

- 已安装Java环境，要求安装JDK 1.8及以上版本。

- 时序引擎为3.4.7及以上版本。

  **说明**

  如何查看或升级当前版本，请参见[时序引擎版本说明](https://help.aliyun.com/zh/lindorm/product-overview/release-notes-of-lindormtsdb#concept-2119016)和[升级小版本](https://help.aliyun.com/zh/lindorm/user-guide/upgrade-the-minor-engine-version-of-an-apsaradb-for-lindorm-instance#concept-2557610)。

- 已将客户端IP地址添加至[Lindorm白名单](https://help.aliyun.com/zh/lindorm/getting-started/configure-a-whitelist#task-2045184)。

- 已获取云原生多模数据库 Lindorm[时序引擎的连接地址](https://help.aliyun.com/zh/lindorm/user-guide/connect-through-lindorm-cli#section-wvz-y1d-uc5)。

## 准备工作

通过Java Native SDK连接Lindorm时序引擎前，需要安装Java Native SDK。以1.0.0版本为例，您可以通过以下三种方式安装Java Native SDK：

- （推荐）在Maven项目中使用Lindorm TSDB Java SDK。在pom.xml文件的`dependencies`中添加以下依赖项。

   

  ```java
  <dependency>
    <groupId>com.aliyun.lindorm</groupId>
    <artifactId>lindorm-tsdb-client</artifactId>
    <version>1.0.4</version>
  </dependency>
  ```

  **说明**

  Lindorm TSDB Java SDK提供了一个基于Maven的示例工程，您可以直接下载[示例工程](https://tsdbtools.oss-cn-hangzhou.aliyuncs.com/lindorm-tsdb-client/lindorm-tsdb-client-demo.zip)并在本地编译和运行，也可以以示例工程为基础开发您的项目工程。

- 在Eclipse项目中导入JAR包。

  1. 下载[Java SDK开发包](https://tsdbtools.oss-cn-hangzhou.aliyuncs.com/lindorm-tsdb-client/lindorm-tsdb-client-latest.zip)。
  2. 解压下载的Java SDK开发包。
  3. 将解压后JAR包添加至Eclipse项目中。
     1. 在Eclipse中打开您的项目，右键单击该项目，选择**Properties**。
     2. 在弹出的对话框中，单击***\*Java Build Path\** > \**Libraries\** > \**Add JARs\****，选择解压后的lindorm-tsdb-client-1.0.0.jar和lib文件中的JAR包。
     3. 单击**Apply and Close**。

- 在IntelliJ IDEA项目中导入JAR包。

  1. 下载[Java SDK开发包](https://tsdbtools.oss-cn-hangzhou.aliyuncs.com/lindorm-tsdb-client/lindorm-tsdb-client-latest.zip)。
  2. 解压下载的Java SDK开发包。
  3. 将解压后JAR包添加至IntelliJ IDEA项目中。
     1. 在IntelliJ IDEA中打开您的项目，在菜单栏单击***\*File\** > \**Project Structure\****。
     2. 在**Project Structure**对话框的左边选择***\*Project Structure\** > \**Modules\****。
     3. 单击右边![添加](assets/p442372.png)，选择**JARs or directories**。
     4. 在弹出的对话框中，选择解压后的lindorm-tsdb-client-1.0.0.jar和lib文件中的JAR包，并单击**OK**。
     5. 单击**Apply**。
     6. 单击**OK**。

**说明**

- Lindorm时序引擎的Java Native SDK各版本可以通过Maven中央仓库获取，更多信息请参见[Maven Repository](https://mvnrepository.com/artifact/com.aliyun.lindorm/lindorm-tsdb-client?spm=a2c4g.11186623.0.0.35fc2897Ue8vnd)。
- Lindorm时序引擎的Java Native SDK各版本说明请参见[版本说明](https://help.aliyun.com/zh/lindorm/user-guide/release-notes#topic-2180790)。

## 操作步骤

1. 创建数据库实例。新建LindormTSDBClient时，需要指定Lindorm[时序引擎的连接地址](https://help.aliyun.com/zh/lindorm/user-guide/connect-through-lindorm-cli#section-wvz-y1d-uc5)。

    

   ```java
   String url = "http://ld-bp17j28j2y7pm****-proxy-tsdb-pub.lindorm.rds.aliyuncs.com:8242";
   // LindormTSDBClient线程安全，可以重复使用，无需频繁创建和销毁
   ClientOptions options = ClientOptions.newBuilder(url).build();
   LindormTSDBClient lindormTSDBClient = LindormTSDBFactory.connect(options);
   ```

2. 创建数据库demo和时序表sensor。关于创建数据库和时序表的SQL语句说明，请参见[CREATE DATABASE](https://help.aliyun.com/document_detail/342954.html#topic-2134218)和[CREATE TABLE](https://help.aliyun.com/document_detail/216795.html#topic-2078307)。

    

   ```java
   lindormTSDBClient.execute("CREATE DATABASE demo");
   lindormTSDBClient.execute("demo","CREATE TABLE sensor (device_id VARCHAR TAG,region VARCHAR TAG,time BIGINT,temperature DOUBLE,humidity DOUBLE,PRIMARY KEY(device_id))");
   ```

3. 在表中写入数据。

   **说明**

   默认情况下，为了提高写入数据的性能，LindormTSDBClient通过异步攒批的方式进行数据写入。如果需要通过同步的方式进行数据写入，可以调用`write()`方法返回的`CompletableFuture<WriteResult>`的`join()`方法。

    

   ```java
   int numRecords = 10;
   List<Record> records = new ArrayList<>(numRecords);
   long currentTime = System.currentTimeMillis();
   for (int i = 0; i < numRecords; i++) {
       Record record = Record
           .table("sensor")
           .time(currentTime + i * 1000)
           .tag("device_id", "F07A1260")
           .tag("region", "north-cn")
           .addField("temperature", 12.1 + i)
           .addField("humidity", 45.0 + i)
           .build();
       records.add(record);
   }
   
   CompletableFuture<WriteResult> future = lindormTSDBClient.write("demo", records);
   // 处理异步写入结果
   future.whenComplete((r, ex) -> {
       // 处理写入失败
       if (ex != null) {
           System.out.println("Failed to write.");
           if (ex instanceof LindormTSDBException) {
               LindormTSDBException e = (LindormTSDBException) ex;
               System.out.println("Caught an LindormTSDBException, which means your request made it to Lindorm TSDB, "
                                  + "but was rejected with an error response for some reason.");
               System.out.println("Error Code: " + e.getCode());
               System.out.println("SQL State:  " + e.getSqlstate());
               System.out.println("Error Message: " + e.getMessage());
           }  else {
               ex.printStackTrace();
           }
       } else  {
           System.out.println("Write successfully.");
       }
   });
   // 这里作为示例, 简单同步处理写入结果
   System.out.println(future.join());
   ```

4. 查询时序表的数据。关于查询操作的SQL语句说明，请参见[基本查询](https://help.aliyun.com/document_detail/216801.html#topic-2078313)。

    

   ```java
   String sql = "select * from sensor limit 10";
   ResultSet resultSet = lindormTSDBClient.query("demo", sql);
   
   try {
       // 处理查询结果
       QueryResult result = null;
       // 查询结果使用分批的方式返回，默认每批1000行
       // 当resultSet的next()方法返回为null，表示已经读取完所有的查询结果
       while ((result = resultSet.next()) != null) {
           List<String> columns = result.getColumns();
           System.out.println("columns: " + columns);
           List<String> metadata = result.getMetadata();
           System.out.println("metadata: " + metadata);
           List<List<Object>> rows = result.getRows();
           for (int i = 0, size = rows.size(); i < size; i++) {
               List<Object> row = rows.get(i);
               System.out.println("row #" + i + " : " + row);
           }
       }
   } finally {
       // 查询结束后，需确保调用ResultSet的close方法，以释放IO资源
       resultSet.close();
   }
   ```

## 完整的代码示例

 

```java
import com.aliyun.lindorm.tsdb.client.ClientOptions;
import com.aliyun.lindorm.tsdb.client.LindormTSDBClient;
import com.aliyun.lindorm.tsdb.client.LindormTSDBFactory;
import com.aliyun.lindorm.tsdb.client.exception.LindormTSDBException;
import com.aliyun.lindorm.tsdb.client.model.QueryResult;
import com.aliyun.lindorm.tsdb.client.model.Record;
import com.aliyun.lindorm.tsdb.client.model.ResultSet;
import com.aliyun.lindorm.tsdb.client.model.WriteResult;
import com.aliyun.lindorm.tsdb.client.utils.ExceptionUtils;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;

public class QuickStart {

    public static void main(String[] args) {

        // 1.创建客户端实例
        String url = "http://ld-xxxx-proxy-tsdb-pub.lindorm.rds.aliyuncs.com:8242";
        // LindormTSDBClient线程安全，可以重复使用，无需频繁创建和销毁
        ClientOptions options = ClientOptions.newBuilder(url).build();
        LindormTSDBClient lindormTSDBClient = LindormTSDBFactory.connect(options);

        // 2.创建数据库demo和表sensor
        lindormTSDBClient.execute("CREATE DATABASE demo");
        lindormTSDBClient.execute("demo","CREATE TABLE sensor (device_id VARCHAR TAG,region VARCHAR TAG,time BIGINT,temperature DOUBLE,humidity DOUBLE,PRIMARY KEY(device_id))");

        // 3.写入数据
        int numRecords = 10;
        List<Record> records = new ArrayList<>(numRecords);
        long currentTime = System.currentTimeMillis();
        for (int i = 0; i < numRecords; i++) {
            Record record = Record
                    .table("sensor")
                    .time(currentTime + i * 1000)
                    .tag("device_id", "F07A1260")
                    .tag("region", "north-cn")
                    .addField("temperature", 12.1 + i)
                    .addField("humidity", 45.0 + i)
                    .build();
            records.add(record);
        }

        CompletableFuture<WriteResult> future = lindormTSDBClient.write("demo", records);
        // 处理异步写入结果
        future.whenComplete((r, ex) -> {
            // 处理写入失败
            if (ex != null) {
                System.out.println("Failed to write.");
                Throwable throwable = ExceptionUtils.getRootCause(ex);
                if (throwable instanceof LindormTSDBException) {
                    LindormTSDBException e = (LindormTSDBException) throwable;
                    System.out.println("Caught an LindormTSDBException, which means your request made it to Lindorm TSDB, "
                            + "but was rejected with an error response for some reason.");
                    System.out.println("Error Code: " + e.getCode());
                    System.out.println("SQL State:  " + e.getSqlstate());
                    System.out.println("Error Message: " + e.getMessage());
                }  else {
                    throwable.printStackTrace();
                }
            } else  {
                System.out.println("Write successfully.");
            }
        });
        // 这里作为示例, 简单同步等待
        System.out.println(future.join());

        // 4.查询数据
        String sql = "select * from sensor limit 10";
        ResultSet resultSet = lindormTSDBClient.query("demo", sql);

        try {
            // 处理查询结果
            QueryResult result = null;
            // 查询结果使用分批的方式返回，默认每批1000行
            // 当resultSet的next()方法返回为null，表示已经读取完所有的查询结果
            while ((result = resultSet.next()) != null) {
                List<String> columns = result.getColumns();
                System.out.println("columns: " + columns);
                List<String> metadata = result.getMetadata();
                System.out.println("metadata: " + metadata);
                List<List<Object>> rows = result.getRows();
                for (int i = 0, size = rows.size(); i < size; i++) {
                    List<Object> row = rows.get(i);
                    System.out.println("row #" + i + " : " + row);
                }
            }
        } finally {
            // 查询结束后，需确保调用ResultSet的close方法，以释放IO资源
            resultSet.close();
        }

        lindormTSDBClient.shutdown();
    }
}
```