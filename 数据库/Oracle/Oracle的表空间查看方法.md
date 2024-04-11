# Oracle表空间
Oracle表空间（tablespaces）是一个逻辑的概念，真正存放数据的是数据文件（data files）。一个Oracle数据库能够有一个或多个表空间,而一个表空间则对应着一个或多个物理的数据库文件。表空间是Oracle数据库恢复的最小单位,容纳着许多数据库实体,如表、视图、索引、聚簇、回退段和临时段等。

1.Oracle表空间的特性:

(1)控制数据库数据磁盘分配；

(2)限制用户在表空间中可以使用的磁盘空间大小；

(3)表空间具有 online, offline, readonly, readwrite属性。

2.表空间的分类：

永久表空间：数据库中要永久化存储的一些对象，如：表、视图、存储过程

临时表空间：数据库操作当中中间执行的过程，执行结束后，存放的内容会被自动释放

UNDO表空间：用于保存事务所修改数据的旧值，可以进行数据的回滚


select * from dba_data_files;   --数据文件信息
select * from dba_temp_files;   -- 临时数据文件信息
select * from dba_free_space;   -- 数据库中所有表空间中的空闲扩展区
select * from dba_segments;     -- 数据库中的所有段分配的存储

1.创建表空间
--语法：
create [temporary] tablespace tablespace_name tempfile|datafile ‘xx.dbf’ size xx;
--创建临时表空间时，加上temporary关键字;
2.扩大表空间，当某个表空间被用完以后，就不能再对数据库表进行insert操作，此时我们需要扩大表空间,即可通过增加datafile文件来扩大表空间。
select f.* from dba_data_files f where f.tablespace_name='MLT';--查看表空间信息
alter tablespace MLT --表空间名
     add datafile '/home/oracle/oradata/crm/mlt04.dbf'  --datafile文件路径
     size 100M --表空间大小
     autoextend on --自动扩展
3.修改表空间的状态
alter tablespace tablespace_name online|offline;--表空间是脱机时不可用，默认是联机的
4.删除表空间
drop tablespace tablespace_name[including contents];
--including contents 表示删除表空间包括datafile数据文件，不加则不删除相关数据文件;
--删除数据文件时，不能删除表空间当中第一个数据文件，如果要删除就需要删除整个表空间。
--查询表空间使用情况
```
SELECT Upper(F.TABLESPACE_NAME)         "表空间名",
       D.TOT_GROOTTE_MB                 "表空间大小(M)",
       D.TOT_GROOTTE_MB - F.TOTAL_BYTES "已使用空间(M)",
       To_char(Round(( D.TOT_GROOTTE_MB - F.TOTAL_BYTES ) / D.TOT_GROOTTE_MB * 100, 2), '990.99')
       || '%'                           "使用比",
       F.TOTAL_BYTES                    "空闲空间(M)",
       F.MAX_BYTES                      "最大块(M)"
FROM   (SELECT TABLESPACE_NAME,
               Round(Sum(BYTES) / ( 1024 * 1024 ), 2) TOTAL_BYTES,
               Round(Max(BYTES) / ( 1024 * 1024 ), 2) MAX_BYTES
        FROM   SYS.DBA_FREE_SPACE
        GROUP  BY TABLESPACE_NAME) F,
       (SELECT DD.TABLESPACE_NAME,
               Round(Sum(DD.BYTES) / ( 1024 * 1024 ), 2) TOT_GROOTTE_MB
        FROM   SYS.DBA_DATA_FILES DD
        GROUP  BY DD.TABLESPACE_NAME) D
WHERE  D.TABLESPACE_NAME = F.TABLESPACE_NAME
```
这个也不赖（查看物理文件名称路径）
```
select
b.file_name 物理文件名,
b.tablespace_name 表空间,
b.bytes/1024/1024 大小M,
(b.bytes-sum(nvl(a.bytes,0)))/1024/1024 已使用M,
substr((b.bytes-sum(nvl(a.bytes,0)))/(b.bytes)*100,1,5) 利用率
from dba_free_space a,dba_data_files b
where a.file_id=b.file_id
group by b.tablespace_name,b.file_name,b.bytes
order by b.tablespace_name
```
--查询表空间的空闲扩展区
select tablespace_name, count(*) AS extends,round(sum(bytes) / 1024 / 1024, 2) AS 大小/MB
,sum(blocks) AS blocks 
from dba_free_space group BY tablespace_name;
--查询表空间的总容量
select tablespace_name, sum(bytes) / 1024 / 1024 as MB 
from dba_data_files group by tablespace_name;
--查询表空间使用率
SELECT total.tablespace_name,
       Round(total.MB, 2)           AS   总量/MB,
       Round(total.MB - free.MB, 2) AS  已使用/MB,
       Round(( 1 - free.MB / total.MB ) * 100, 2) || '%'                  AS  使用率
FROM   (SELECT tablespace_name,
               Sum(bytes) / 1024 / 1024 AS MB
        FROM   dba_free_space
        GROUP  BY tablespace_name) free,
       (SELECT tablespace_name,
               Sum(bytes) / 1024 / 1024 AS MB
        FROM   dba_data_files
        GROUP  BY tablespace_name) total
WHERE  free.tablespace_name = total.tablespace_name;
