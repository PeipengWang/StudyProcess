# show full processlist状态详解
Id: 连接或线程的标识符，是一个唯一的数字。
User: 连接使用的用户名。
Host: 连接来自的主机名或 IP 地址。
db: 当前连接使用的数据库名称，如果没有选择任何数据库则为 NULL。
Command: 当前连接正在执行的 MySQL 命令，例如 Query、Sleep、Connect 等。
Time: 当前命令执行的时间（秒）。
State: 当前连接的状态信息，例如 Copying to tmp table、Locked、Sending data 等。
Info: 当前连接正在执行的 SQL 语句，如果没有执行任何语句则为 NULL。
Progress: 当前操作的进度信息，如果没有进度则为 NULL。
Rows_sent: 已经发送到客户端的行数。
Rows_examined: 已经扫描的行数。
Rows_affected: 已经修改或删除的行数。
Timestamp: 当前时间戳。

status详解：
init: 表示当前连接正在初始化或准备执行操作。
Waiting for table metadata lock: 表示当前连接正在等待元数据锁，通常发生在 ALTER TABLE 或 DROP TABLE 等语句执行时。
Creating sort index: 表示当前连接正在创建排序索引。
Copying to tmp table: 表示当前连接正在将数据复制到临时表中，通常发生在 GROUP BY 或 ORDER BY 等操作执行时。
Sorting result: 表示当前连接正在对结果进行排序。
Sending data: 表示当前连接正在将数据发送到客户端。
Locked: 表示当前连接正在等待锁。
Updating: 表示当前连接正在执行 UPDATE 操作。
Deleting: 表示当前连接正在执行 DELETE 操作。
Selecting: 表示当前连接正在执行 SELECT 操作。
Analyzing: 表示当前连接正在分析查询，以确定最佳查询计划。
Repair by sorting: 表示当前连接正在对表进行排序，以修复表中的数据。
Repair with keycache: 表示当前连接正在使用索引缓存来修复表中的数据。
Repair by keycache: 表示当前连接正在使用索引缓存来修复表中的数据。
Repair with sort: 表示当前连接正在使用排序来修复表中的数据。

Sleep：线程正在等待客户端发送下新的请求
Query：线程正在执行查询或者正在将结果发给客户端
Locked：在Mysql服务层，该线程正在等待表锁，在存储引擎级别实现的锁，例如InnoDB的行锁，并不会体现在线程状态中
Analyzing and statistics：线程正在收集存储引擎的统计信息，并生成查询的执行计划
Copying to tmp table [on disk]: 线程正在执行查询，并且将结果集都复制到另一个临时表中，这种状态一般是要么在做group by 操作，要么是文件排序或者是union操作。如果后面跟着一个on disk标记，那表示Mysql在将内存临时表放到磁盘上
Sorting result：线程在对结果集进行排序
Sending data：在多个状态传送数据
