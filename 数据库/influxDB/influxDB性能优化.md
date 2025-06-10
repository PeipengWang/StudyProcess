# 数据写入方案优化

1、批量写入数据

在批量写入数据时，可以最大程度减少网络开销，InfluxDB建议最佳批量数据大小为5000行协议

2、按key值排序tag

写入数据时，尽可能的对tag进行排序

3、尽可能使用最粗略的时间精度

# 数据查询优化方案

1、使用下推启动查询

所谓下推，就是将数据推送到底层数据源而不是对内存中的数据进行操作函数或者函数组合，其作用是提高查询性能，一旦非下推函数运行，FLux会将数据拉入内存中，并在那里运行所有后续操作。

2、合理使用set和map函数

set()函数为写入表中的每条数据分配一个静态值。

map()函数将函数应用于输入表中的每条记录。

set()函数在性能上优于map()，因此在设置为预定义的静态值时要用set()函数，如果过使用现有列行数据动态设置列值则使用map()

3、谨慎使用功能较为繁琐的函数

# 性能测试

## 1、安装go环境

## 2、获取测试安装工具

```
go get -v github.com/influxdata/influx-stress/cmd/..
```

## 3、进行测试

# 数据保留策略

## 创建保留策略

使用CREATE RETENTION POLICY创建保留策略



CREATE RETENTION POLICY  <retention_policy_name> ON  <database_name> DURATION <duration> REPLICATION <n> [SHARED DURATION <duration>]

retention_policy：保留策略名称

database_name：要操作的数据库

DURATION：标识持续时间。数据保留的时长，最短保留时间时1小时，最长时无限INF

例如：

```sql
CREATE RETENTION POLICY "rp_create" ON "NOAA_water_database" DURATION 1h REPLICATION 1
```

创建一个名称为rp_create的保留策略，放到NOAA_water_database库中，保留时间为1小时

## 查询保留策略

```
SHOW RETENTION POLICIES ON <database_name>
```

查询database_name库中的保留策略

## 删除保留策略

```
DROP RETENTION POLICY <retention_policy_name> ON <database_name>
```

## 修改保留策略

```
ALTER RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n>
```

# influxDB基本操作与写入查询

## 写入数据

```
influx write -b bucket-name -o group_name -p s --format-csv  -f path
influx write：写入命令
-b： 写入哪个数据桶
-o: 后面填写桶所在的组织
-p： 设置精度
--format：导入文件的格式，如果时csv表格则写csv
-f：文件路径
```

## 行协议

在插入数据到InfluxDB时，要指定把数据插入哪个桶和表中，这种插入时要遵守的语法格式就是InfluxDB行协议。行协议时写入数据时要遵守的数据格式，只有遵守这个协议的语句，才能被InfluxDB正确识别写入。

行协议写入实例

```
influx write --org goup_name --bucket bucket_name "m,host=host1 field=1.0"
```

双引号中就是要插入的数据，也就是一个point

行协议要素：

```
measurement,tag_key=tag_value fied_key=field_value timestamp
```

## 桶操作

### 创建桶

```
influx bucket create --name mydb
```

### 显示桶

```
influx bucket list
```

### 删除桶

```
influx bucket delete --name mydb
```

### 修改桶

```
influx bucket update --ID 78898788sadn --name new_bucket_name
```

## 查询操作

查询操作与sql差不多，这里只罗列与sql不同的地方

### 类型转换

```
select field_key::type  from measurement_name
```

例如

```
select field_key::float from measurement_name
```

查询measurement_name表中的fiekd_key字段数据，并且强制转换成float类型输出

### where子句

支持where子句的判断条件

### 函数聚合

count() 返回非空字段值的数量，可以嵌套distinct()

```
select count("water_level") from "h2o_feet"
```

sum() 返回字段和的平均值

stddev() 返回字段值的标准差

mode() 返回出现频率最高的指标

median() 返回中位数

mean() 返回字段平均值

integral() 返回字段值曲线下的面积，即积分

top() 返回最大的前n个字段

sample() 返回n个随机抽样字段

percentile() 返回百分位数为n的字段值

max(),min()

latest() 返回时间戳最新的字段

first() 返回时间戳最早的字段

bottom() 返回最小的n个字段值

### group by子句

**group by子句可以按照标签进行分组**

可以按照时间间隔进行分组 

```
group by time(time_interval)
```

### 其它

limit和slimit

排序 orderby

offset和soffset  跳过前n条数据



