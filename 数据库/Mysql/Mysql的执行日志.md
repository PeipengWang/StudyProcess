Mysql的执行日志可以帮助我们确定在项目中是否完了了连接数据库，尤其是在生产环境中，直接使用命令方式进行定位能够解决很多问题，下面介绍几个常用命令：
**准备：确定日志开启状态**
首先确认你日志是否启用了mysql>show variables like 'log_bin'。
如果启用了，即ON，那日志文件就在mysql的安装目录的data目录下。
怎样知道当前的日志mysql> show master status。
看二进制日志文件用mysqlbinlog，shell>mysqlbinlog mail-bin.000001或者shell>mysqlbinlog mail-bin.000001 | tail，Windows 下用类似的。
**基本操作**
1、查看正在执行的SQL
-- 切换数据库

```
use information_schema;
```

-- 查看正在执行的SQL语句

```
show processlist;
```

-- 或者直接使用SQL语句查询

```
select * from information_schema.PROCESSLIST where info is not null;
```

2、开启日志模式，记录所有SQL语句执行记录
-- 查看当前日志输出类型：table / file ，可根据需要具体设置

```
show variables like 'log_output';
```

-- 设置日志输出至table

```
set global log_output='table';
```

-- 设置日志输出至file

```
set global log_output='file';
```

-- 查看日志输出文件的保存路径

```
show variables like 'general_log_file';
```

-- 修改日志输出文件的保存路径

```
set global general_log_file='tmp/general.log';
```

-- 完成日志模式设置后，开启日志功能
-- 查看日志功能设置状态

```
show variables like 'general_log';
```

-- 打开日志记录功能

```
set global general_log=on;
```

-- 关闭日志记录功能

```
set global general_log=off;
```

-- 日志输出至table模式，查看日志记录

```
SELECT * from mysql.general_log ORDER BY event_time DESC;
```

-- 日志输出至table模式，清空日志记录

```
truncate table mysql.general_log;
```

-- 日志输出至file模式，查看日志记录

```
cat /tmp/general.log
```
**查看数据库连接用户**
设置连接数
方法一：进入MySQL用命令行修改，但是MySQL重启后就失效了，需要重新设置。（不推荐）
　　命令如下：

　　1、show variables like 'max_connections';（查看当前最大连接数）
　　2、set global max_connections=1000;（设置最大连接数为1000，可以再次执行上面的命令查看设

置是否成功）
　　方法二：修改MySQL配置文件，永久生效（推荐）
　　1、进入MySQL安装目录，打开my.ini或my.cnf文件；
　　2、查看max_connections=100的配置参数，修改为max_connections=1000；如果没有这个参数，直接添加max_connections=1000即可；
　　3、保存配置文件并退出，重启MySQL服务即可。
1、查看数据库当前连接信息,可以看到连接数据库的进程id，ip,用户名，连接的数据库，连接状态，连接时长等
　　命令：`SHOW FULL processlist`
