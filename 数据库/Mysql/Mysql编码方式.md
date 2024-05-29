# mysql编码方式
 SHOW CREATE TABLE zxinsys.oper_information2;  查看某张表的编码方式  
 show variables like 'character%';  
 show variables like "character%";  
+--------------------------+----------------------------+  
| Variable_name            | Value                      |  
+--------------------------+----------------------------+  
| character_set_client     | latin1                      |  
| character_set_connection | latin1                       |  
| character_set_database   | latin1                     |  
| character_set_filesystem | binary                     |  
| character_set_results    | latin1                      |  
| character_set_server     | latin1                     |  
| character_set_system     | latin1                      |  
| character_sets_dir       | /opt/lampp/share/charsets/ |  
从以上信息可知：  
数据库的编码为latin1  
character_set_client为客户端编码方式；  
character_set_connection为建立连接使用的编码；  
character_set_database数据库的编码；  
character_set_results结果集的编码；
character_set_server数据库服务器的编码；  
修改编码方式  
修改mysql的配置文件my.ini，该文件目录一般为/etc/my.ini  
找到客户端配置[client] 在下面添加  

```  
default-character-set=utf8 
```

默认字符集为utf8  

在找到[mysqld] 添加  

```
default-character-set=utf8
```

默认字符集为utf8  
```
init_connect='SET NAMES utf8'
```

（设定连接mysql数据库时使用utf8编码，以让mysql数据库为utf8运行）   

