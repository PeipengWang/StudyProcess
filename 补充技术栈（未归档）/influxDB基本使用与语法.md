在 Java 中操作 InfluxDB（一个高性能的时序数据库）可以使用 InfluxDB 的 Java 客户端库。例如，`influxdb-java` 是一个常用的库。下面是一个简单的示例，展示如何在 Java 中使用这个库进行基本的增、删、查、改操作。

首先，你需要添加 Maven 依赖：

```xml
<dependency>
    <groupId>org.influxdb</groupId>
    <artifactId>influxdb-java</artifactId>
    <version>2.21</version>
</dependency>
```

然后，以下是具体的代码示例：

### 1. 连接到 InfluxDB

```java
import org.influxdb.InfluxDB;
import org.influxdb.InfluxDBFactory;

public class InfluxDBExample {
    public static void main(String[] args) {
        // 连接到 InfluxDB
        InfluxDB influxDB = InfluxDBFactory.connect("http://localhost:8086", "username", "password");
        // 设置数据库
        influxDB.setDatabase("mydb");
    }
}
```

### 2. 插入数据

```java
import org.influxdb.dto.Point;

import java.util.concurrent.TimeUnit;

public class InfluxDBExample {
    public static void main(String[] args) {
        // 连接到 InfluxDB
        InfluxDB influxDB = InfluxDBFactory.connect("http://localhost:8086", "username", "password");
        influxDB.setDatabase("mydb");

        // 插入数据
        Point point = Point.measurement("cpu")
                .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
                .addField("host", "server01")
                .addField("region", "us-west")
                .addField("value", 0.64)
                .build();

        influxDB.write(point);
    }
}
```

### 3. 查询数据

```java
import org.influxdb.dto.Query;
import org.influxdb.dto.QueryResult;

public class InfluxDBExample {
    public static void main(String[] args) {
        // 连接到 InfluxDB
        InfluxDB influxDB = InfluxDBFactory.connect("http://localhost:8086", "username", "password");
        influxDB.setDatabase("mydb");

        // 查询数据
        Query query = new Query("SELECT * FROM cpu WHERE host = 'server01'", "mydb");
        QueryResult result = influxDB.query(query);

        // 处理查询结果
        for (QueryResult.Result res : result.getResults()) {
            for (QueryResult.Series series : res.getSeries()) {
                System.out.println(series.getName());
                for (List<Object> values : series.getValues()) {
                    System.out.println(values);
                }
            }
        }
    }
}
```

### 4. 更新数据

InfluxDB 不直接支持更新数据，因为它是时序数据库，数据是不可变的。通常的做法是插入新的数据点来覆盖旧的数据点。例如，如果你想更新某个时间点的数据，可以插入一个相同时间戳的新数据点。

### 5. 删除数据

```java
public class InfluxDBExample {
    public static void main(String[] args) {
        // 连接到 InfluxDB
        InfluxDB influxDB = InfluxDBFactory.connect("http://localhost:8086", "username", "password");
        influxDB.setDatabase("mydb");

        // 删除数据
        String deleteQuery = "DELETE FROM cpu WHERE host = 'server01'";
        influxDB.query(new Query(deleteQuery, "mydb"));
    }
}
```

### 6. 关闭连接

```java
public class InfluxDBExample {
    public static void main(String[] args) {
        // 连接到 InfluxDB
        InfluxDB influxDB = InfluxDBFactory.connect("http://localhost:8086", "username", "password");
        influxDB.setDatabase("mydb");

        // 其他操作...

        // 关闭连接
        influxDB.close();
    }
}
```

通过上述代码，你可以在 Java 中进行基本的 InfluxDB 操作，包括插入、查询和删除数据。请确保你已经在本地或服务器上安装并运行 InfluxDB，并且已创建相关的数据库（例如 `mydb`）。





```xml
<dependency>
    <groupId>com.influxdb</groupId>
    <artifactId>influxdb-client-java</artifactId>
    <version>6.7.0</version>
    <exclusions>
        <exclusion>
            <artifactId>okhttp</artifactId>
            <groupId>com.squareup.okhttp3</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <artifactId>okhttp</artifactId>
    <groupId>com.squareup.okhttp3</groupId>
    <version>4.10.0</version>
</dependency>


```

[InfluxDB](https://so.csdn.net/so/search?q=InfluxDB&spm=1001.2101.3001.7020)的API本质上就是通过HTTP请求到InfluxDB进行操作,所以他的API根据不同的功能分成了不同API类型 具体需要操作某一种资源 选择具体API即可

常用的API

- WriteApiBlocking 同步写入API WriteApi 异步写入API
- DeleteApi 删除API
- QueryApi 同步查询API InfluxQLQueryApi SQL查询API

```java
public class InfluxApiTest {

    final static String token = "DMq0mfKS2-Qy9_cr8bmKdPcX-oA-R55WmOZ8VDQP0SuUoKb1bOPZZa2auS5oYr8u8KiupXol_R2Km3TeJ7IAXA==";

    final static String bucket = "java_api_bucket";

    final static String org = "corn";

    final static String url = "http://192.168.198.129:8086";

    static InfluxDBClient client;

    @BeforeAll
    public static void initClient() {
        client = InfluxDBClientFactory.create(url, token.toCharArray(), org, bucket);
    }

    @AfterAll
    public static void closeClient() {
        client.close();
    }

    /**
     * 同步写入API WriteApiBlocking
     * client.getWriteApiBlocking()
     */
    @Test
    public void testInsertData() {
        WriteApiBlocking writeApiBlocking = client.getWriteApiBlocking();
        // 1. 通过Influx 行协议的方法写入 writeRecord
        // 选择时间戳模式
        writeApiBlocking.writeRecord(WritePrecision.MS, "temperature,city=bj value=34.2");

        // 2. 通过写入 Point点API
        Point point = Point.measurement("temperature")
                .addTag("city", "sh")
                .addField("value", 32.12);
        writeApiBlocking.writePoint(point);

        // 3. POJO的模式写入
        Temperature temperature = new Temperature();
        temperature.setCity("sc");
        temperature.setValue(40.1);
        writeApiBlocking.writeMeasurement(WritePrecision.MS, temperature);
    }

    /**
     * 异步写入API
     * 1. 默认INFLUX 在将请求存放在缓冲区 每1s(可调整) 或者 每批次数据大于1000条(可调整) 才会一次性发送给Influx Server
     * 2. 调用flush会在强制发送缓冲区的请求
     * 3. 调用close会强制发送缓冲区数据
     * 4. 其他写入API 与同步写 一样 可以使用 行协议 Point 或者 POJO
     */
    @Test
    public void testAysnInserData() {
        WriteOptions options = WriteOptions.builder()
                // 每批次500条
                .batchSize(500)
                // 每2s 刷写一次
                .flushInterval(2000)
                .build();
        WriteApi writeApi = client.makeWriteApi(options);

        Temperature temperature = new Temperature();
        temperature.setCity("hk");
        temperature.setValue(23.1);
        writeApi.writeMeasurement(WritePrecision.MS, temperature);

        writeApi.flush();

    }

    /**
     * 删除某一时间窗口指定的数据
     */
    @Test
    public void testDelete() {

        /**
         *  只能通过tag进行操作
         *  "tag1=\"value1\" and (tag2=\"value2\" and tag3!=\"value3\")"
         *
         */
        String predictSql = "city=\"hk\"";
        DeleteApi deleteApi = client.getDeleteApi();
        deleteApi.delete(OffsetDateTime.now().minusHours(12),
                OffsetDateTime.now(),
                predictSql,
                bucket,
                org
        );
    }

    @Test
    public void testQuery(){
        QueryApi queryApi = client.getQueryApi();
        String queryFlux =
                "from(bucket: \"java_api_bucket\")\n" +
                "  |> range(start: -12h)\n" +
                "  |> filter(fn: (r) => r[\"_measurement\"] == \"temperature\")\n" +
                "  |> filter(fn: (r) => r[\"_field\"] == \"value\")\n" +
                "  |> filter(fn: (r) => r[\"city\"] == \"sh\")\n" +
                "  |> aggregateWindow(every: 5s, fn: mean, createEmpty: false)\n" +
                "  |> yield(name: \"mean\")";
        List<Temperature> list = queryApi.query(queryFlux, Temperature.class);
        Assertions.assertNotNull(list);
    }


    /**
     * SQL的方式查询
     */
    @Test
    public void testQueryBySQL(){
        InfluxQLQueryApi sqlApi = client.getInfluxQLQueryApi();
        String sql = "SELECT value FROM temperature WHERE city = 'sh'";
        List<Temperature> target = new ArrayList<>();
        InfluxQLQuery query = new InfluxQLQuery(sql,bucket);
        InfluxQLQueryResult result = sqlApi.query(query, (columnName, rawValue, resultIndex, seriesName) -> {
            Temperature temperature = new Temperature();
            if(columnName.equals("value")){
                temperature.setValue(Double.valueOf(rawValue));
                target.add(temperature);
            }
            return temperature;
        });
        result.getResults().forEach(r->{
            r.getSeries().forEach(s->{
                System.out.println(s.getColumns());
            });
        });
        System.out.println("===========================================");
        target.forEach(System.out::println);
    }
}

```

