## 步骤一：下载

```shell
wget https://tsdbtools.oss-cn-hangzhou.aliyuncs.com/lindorm-cli-linux-latest.tar.gz
```

arm64

```shell
wget https://tsdbtools.oss-cn-hangzhou.aliyuncs.com/lindorm-cli-linux-arm64-latest.tar.gz
```

## **步骤二：连接Lindorm时序引擎**

```shell
lindorm-cli -url <Lindorm时序SQL地址> -username <用户名> -password <密码> -database <目标数据库名>
```

查看 表空间

```
 show databases;
+--------------------+
| Database           |
+--------------------+
| default            |
| information_schema |
+--------------------+
```

## **步骤三：使用Lindorm时序引擎**

### **建表**

1. 创建时序数据表。

   ```sql
   CREATE TABLE sensor (
       device_id VARCHAR NOT NULL,
       region VARCHAR NOT NULL,
       time TIMESTAMP NOT NULL,
       temperature DOUBLE,
       humidity BIGINT,
       PRIMARY KEY(device_id, region, time)
   );
   ```

查看表sensor是否创建成功。

```sql
SHOW TABLES;
```

```
use default;
Using database default
Lindorm:default> CREATE TABLE sensor (
    >     device_id VARCHAR NOT NULL,
    >     region VARCHAR NOT NULL,
    >     time TIMESTAMP NOT NULL,
    >     temperature DOUBLE,
    >     humidity BIGINT,
    >     PRIMARY KEY(device_id, region, time)
    > );
0 row affected (107 ms)
Lindorm:default> show tables;
+-------------------+
| Tables_In_default |
+-------------------+
| sensor            |
+-------------------+
1 rows in set (63 ms)

```

查看时序数据表的字段信息。

```sql
DESCRIBE TABLE sensor;
```

### **数据写入**

**说明**

如果TAGS相同，并且时间戳列相同，那么数据被认为是同一条数据，后写入的数据会覆盖先写入的数据。

- 单条依次写入。

  ```sql
  INSERT INTO sensor (device_id, region, time, temperature, humidity) VALUES('F07A1260','north-cn','2021-04-22 15:33:00',12.1,45);
  INSERT INTO sensor (device_id, region, time, temperature, humidity) VALUES('F07A1260','north-cn','2021-04-22 15:33:10',13.2,47);
  INSERT INTO sensor (device_id, region, time, temperature, humidity) VALUES('F07A1260','north-cn','2021-04-22 15:33:20',10.6,46);
  INSERT INTO sensor (device_id, region, time, temperature, humidity) VALUES('F07A1261','south-cn','2021-04-22 15:33:00',18.1,44);
  INSERT INTO sensor (device_id, region, time, temperature, humidity) VALUES('F07A1261','south-cn','2021-04-22 15:33:10',19.7,44);
  ```

- 批量写入。

   

  ```sql
  INSERT INTO sensor (device_id, region, time, temperature, humidity) 
  VALUES ('F07A1260','north-cn','2021-04-22 15:33:00',12.1,45),
         ('F07A1260','north-cn','2021-04-22 15:33:10',13.2,47),
         ('F07A1260','north-cn','2021-04-22 15:33:20',10.6,46),
         ('F07A1261','south-cn','2021-04-22 15:33:00',18.1,44),
         ('F07A1261','south-cn','2021-04-22 15:33:10',19.7,44);
  ```

### **数据查询**

查询设备`F07A1260`在时间范围`2021-04-22 15:33:00`和`2021-04-22 15:33:20`之间的数据：

```sql
SELECT device_id,region,time,temperature,humidity FROM sensor WHERE device_id = 'F07A1260' AND time >= '2021-04-22 15:33:00' AND time <= '2021-04-22 15:33:20';
```

查询结果如下：

```shell
+-----------+----------+---------------------------+-------------+----------+
| device_id |  region  |           time            | temperature | humidity |
+-----------+----------+---------------------------+-------------+----------+
| F07A1260  | north-cn | 2021-04-22T15:33:00+08:00 | 12.1        | 45       |
| F07A1260  | north-cn | 2021-04-22T15:33:10+08:00 | 13.2        | 47       |
| F07A1260  | north-cn | 2021-04-22T15:33:20+08:00 | 10.6        | 46       |
+-----------+----------+---------------------------+-------------+----------+
```

## Lindorm-cli常用命令

- `help`：查看帮助命令。
- `connect`：连接服务器命令。
- `precision`：时间显示格式，支持rfc3339、h、m、s、ms、u或ns。
- `exit`或者`quit`：退出当前Lindorm时序引擎的连接。

```shell
./lindorm-cli -url jdbc:lindorm:tsdb:url=http://ld-m5eti8gl11x7b02jw-proxy-tsdb-pub.lindorm.rds.aliyuncs.com:8242 -username user -password test -database default
```

## **常见问题**

### **数据插入成功后，查询结果为空是什么原因？**

写入数据后如果查询数据的结果为空，请使用`DESCRIBE DATABASE <数据库名称>`语句查询数据库是否设置了数据保存有效期（TTL），如果数据超过保存有效期会被自动清理并且无法查询。