# mysql数据库备份与恢复 
## mysqldump工具备份  
备份整个数据库  
  
$> mysqldump -u root -h host -p dbname > backdb.sql
备份数据库中的某个表  

$> mysqldump -u root -h host -p dbname tbname1, tbname2 > backdb.sql
备份多个数据库

$> mysqldump -u root -h host -p --databases dbname1, dbname2 > backdb.sql
备份系统中所有数据库  

$> mysqldump -u root -h host -p --all-databases > backdb.sql  
还原：  
mysql命令导入sql文件还原  
$> mysql -u root -p [dbname] < backup.sql  
执行前需要先创建dbname数据库，如果backup.sql是mysqldump创建的备份文件则执行是不需要dbname。  
MYSQL> source backup.sql;    
执行source命令前需要先选择数据库。    
## mysqlhotcopy工具备份  
备份数据库或表最快的途径，只能运行在数据库目录所在的机器上，并且只能备份MyISAM类型的表。  

要使用该备份方法必须可以访问备份的表文件。

$> mysqlhotcopy -u root -p dbname /path/to/new_directory;  
#将数据库复制到new_directory目录。   
mysqlhotcopy快速恢复    
停止mysql服务，将备份数据库文件复制到存放数据的位置（mysql的data文件夹），重先启动mysql服务即可(可能需要指定数据库文件的所有者)。    

$> cp -R /usr/backup/test /usr/local/mysql/data  
 如果恢复的数据库已经存在，则使用DROP语句删除已经存在的数据库之后，恢复才能成功，还需要保证数据库版本兼容  
直接复制整个数据库目录(对于InnoDB存储引擎不适用)备份  
