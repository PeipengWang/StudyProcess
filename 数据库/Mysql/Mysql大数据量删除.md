# Mysql大数据量删除
在一些操作中，可能需要清理一下积压的数据，如果数据量小的话自然没有问题，但是如果是个大数据量的问题，那么就该考虑一个合适的办法了。
在清理大数据量的时候需要考虑是清理部分数据还是清理所有数据，这两种场景有着不同的策略。
注意：本次测试与方法均针对mysql5.7，存储引擎为InnoDB

## 清理表中的所有数据
清空表数据，建议直接使用truncate，效率上truncate远高于delete，truncate不走事务，不会锁表，也不会产生大量日志写入日志文件，我们访问log执行日志可以发现每次delete都有记录。truncate table table_name 会立刻释放磁盘空间，并重置auto_increment的值，delete 删除不释放磁盘空间，insert会覆盖之前的数据上，因为我们创建表的时候有一个创建版本号。
delete删除数据的原理:(delete属于DML语句)
表中的数据被删除了，但是这个数据在硬盘上的真实存储空间不会被释放！！！
这种删除表的优点是：支持回滚，后悔了可以恢复数据，可以删除单条数据
缺点：删除效率比较低

效率比较高，表被一次截断，物理删除
优点：快速，不走事务，不会锁表，也不会产生大量日志写入日志文件
缺点：不支持回滚，只能删除表中所有数据，不能删单条数据
如果说公司项目里面有一张大表，数据非常多，几亿条记录：
删除的时候，使用delete，也许执行一个小时才能删除完，效率极其低；
可以选择使用truncate删除表中的数据。只需要不到1s的时间就能删除结束，效率较高。
但是使用truncate之前，必须仔细询问客户是否真的需要删除，并警告删除之后不可恢复！！！

删除表操作：
drop table 表名；// 删除表，不是删除表中的数据

## 清理表中部分数据
### 情景一：如果删除的数据占据表的绝大部分
这是mysql官方文档中提到的一种情形，这里直接复制过来
https://dev.mysql.com/doc/refman/8.0/en/delete.html
如果要从大型表中删除许多行，则可能会超出表的锁定表大小InnoDB。为了避免这个问题，或者只是为了最大限度地减少表保持锁定的时间，以下策略（根本不使用 DELETE）可能会有所帮助：
选择不需要删除的行到一个与原表结构相同的空表中：
INSERT INTO t_copy SELECT * FROM t WHERE ... ;
用于RENAME TABLE以原子方式将原始表移开并将副本重命名为原始名称：
RENAME TABLE t TO t_old, t_copy TO t;
删除原始表：
DROP TABLE t_old;
总体来说就是：**建立一个相同的表，把不删除得数据复制的新表，然后将表重命名倒换，最后删掉旧表**

### 情形二：数据是主键索引
删除大表的多行数据时，会超出innod block table size的限制，最小化的减少锁表的时间的方案是：
1、选择不需要删除的数据，并把它们存在一张相同结构的空表里
2、重命名原始表，并给新表命名为原始表的原始表名
3、删掉原始表
### 每次删除固定的数据量
批量删除（每次限定一定数量），然后循环删除直到全部数据删除完毕；同时key_buffer_size 由默认的8M提高到512M
DELETE FROM test_table WHERE value=12;
如果要用order by 必须要和 limit 联用，否则被优化掉。然后分多次执行就可以把这些记录成功删除。
注意：
执行大批量删除的时候注意要使用上limit。因为如果不用limit，删除大量数据很有可能造成死锁。
如果delete的where语句不在索引上，可以先找主键，然后根据主键删除数据库。
平时update和delete的时候最好也加上limit 1 来防止误操作。
### 暂时删除索引
在My SQL数据库使用中，有的表存储数据量比较大，达到每天三百万条记录左右，此表中建立了三个索引，这些索引都是必须的，其他程序要使用。由于要求此表中的数据只保留当天的数据，所以每当在凌晨的某一时刻当其他程序处理完其中的数据后要删除该表中昨天以及以前的数据，使用delete删除表中的上百万条记录时，MySQL删除速度非常缓慢，每一万条记录需要大概4分钟左右，这样删除所有无用数据要达到八个小时以上，这是难以接受的。
查询MySQL官方手册得知删除数据的速度和创建的索引数量是成正比的，于是删除掉其中的两个索引后测试，发现此时删除速度相当快，一百万条记录在一分钟多一些，可是这两个索引其他模块在每天一次的数据整理中还要使用，于是想到了一个折中的办法：
在删除数据之前删除这两个索引，此时需要三分钟多一些，然后删除其中无用数据，此过程需要不到两分钟，删除完成后重新创建索引，因为此时数据库中的数据相对较少，约三四十万条记录(此表中的数据每小时会增加约十万条)，创建索引也非常快，约十分钟左右。这样整个删除过程只需要约15分钟。对比之前的八个小时，大大节省了时间。
### 强制指定索引
### 分表
如果数据量过大，可以考虑分表，这个分表策越需要根据实际情况来决定，比如每月建立一个表，这个表只存储当月的数据，下个月之后直接将此表truncate。
### 表分区，直接删除过期日期所在的分区
官方文档 https://dev.mysql.com/doc/refman/5.7/en/alter-table-partition-operations.html
 MySQL表分区有几种方式，包括RANGE、KEY、LIST、HASH，详情请参见官方文档。应用场景：日期在变化，所以不适合用RANGE设置固定的分区名称，HASH分区更符合此处场景
分区表定义，SQL语句如下：
ALTER TABLE table_name PARTITION BY HASH(TO_DAYS(cnt_date)) PARTITIONS 7;
TO_DAYS将日期（必须为日期类型，否则会报错:Constant, random or timezone-dependent expressions in (sub)partitioning function are not allowed）转换为天数（年月日总共的天数），然后HASH；建立7个分区。实际上，就是 days MOD 7 。
### 异步删除

## 前置数据
在这之前首先要建立一个存储过程可表来做测试
建立一个表：
```
CREATE TABLE test_table (
    starttime DATETIME,
    endtime DATETIME,
    resourceid INT,
    value INT,
    PRIMARY KEY (resourceid),
    INDEX idx_starttime_endtime_resourceid (starttime, endtime, resourceid)
);
```
定义了主键 resourceid，通过 PRIMARY KEY 关键字指定。

然后，我们使用 INDEX 关键字创建了一个名为 idx_starttime_endtime_resourceid 的联合索引，该索引包含了 starttime、endtime 和 resourceid 列。注意，INDEX 关键字在MySQL中用于创建普通索引。

```
DELIMITER //

CREATE PROCEDURE insert_data(IN num_records_to_generate INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE start_time DATETIME DEFAULT '2023-06-29 00:00:00';

    WHILE i <= num_records_to_generate DO
        INSERT INTO test_table (starttime, endtime,   value)
        VALUES (start_time, DATE_ADD(start_time, INTERVAL 1 SECOND), 12);

        SET start_time = DATE_ADD(start_time, INTERVAL 1 SECOND);
        SET i = i + 1;
    END WHILE;
END //

DELIMITER ;

```
在这个存储过程中，是以endtime作为变量来测试的。
调用方式为
call inser_data(插入数目)

```
mysql> call insert_data(1000);
Query OK, 1 row affected (4.18 sec)
mysql> select count(*) from test_table;
+----------+
| count(*) |
+----------+
|     1000 |
+----------+
1 row in set (0.00 s
```

## 引用文献
https://blog.csdn.net/jike11231/article/details/126551510
https://www.cnblogs.com/NaughtyCat/p/one-fast-way-to-delete-huge-data-in-mysql.html







