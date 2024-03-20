# Mysql主从复制
## Mysql复制的应用
MySQL复制（Replication）是一种用于在多个MySQL服务器之间复制数据的机制。这种机制通过将更改从一个MySQL数据库服务器传递到其他MySQL服务器，从而保持多个服务器上的数据一致性。  
1、读写分离： 通过在主服务器上执行写操作，然后将这些更改复制到多个从服务器上，可以实现读写分离。主服务器处理写入操作，而从服务器用于处理读取操作，从而提高性能和扩展性。  
2、数据备份： 复制可以用作数据备份的一种形式。通过将主服务器的数据复制到一个或多个从服务器上，可以创建数据的实时备份，以应对数据损坏、误删除等情况。  
3、负载均衡： 将读取请求分发到多个从服务器上，以平衡主服务器上的负载，提高整体性能。  
4、高可用性： 复制可以提高系统的可用性。如果主服务器发生故障，可以将一个从服务器提升为新的主服务器，以确保系统持续运行。  
## Mysql复制的原理
MySQL 主从复制是指数据可以从一个MySQL数据库服务器主节点复制到一个或多个从节点。MySQL 默认采用异步复制方式，这样从节点不用一直访问主服务器来更新自己的数据，数据的更新可以在远程连接上进行，从节点可以复制主数据库中的所有数据库或者特定的数据库，或者特定的表。  
原理：  
 （1）master服务器将数据的改变记录二进制binlog日志，当master上的数据发生改变时，则将其改变写入二进制日志中；  
 （2）slave服务器会在一定时间间隔内对master二进制日志进行探测其是否发生改变，如果发生改变，则开始一个I/OThread请求master二进制事件  
 （3）同时主节点为每个I/O线程启动一个dump线程，用于向其发送二进制事件，并保存至从节点本地的中继日志中，从节点将启动SQL线程从中继日志中读取二进制日志，在本地重放，使得其数据和主节点的保持一致，最后I/OThread和SQLThread将进入睡眠状态，等待下一次被唤醒。  
也就是说：  
从库会生成两个线程,一个I/O线程,一个SQL线程;  
I/O线程会去请求主库的binlog,并将得到的binlog写到本地的relay-log(中继日志)文件中;  
主库会生成一个log dump线程,用来给从库I/O线程传binlog;  
SQL线程,会读取relay log文件中的日志,并解析成sql语句逐一执行;  

## docker安装mysql
查询可用版本  
docer search  
拉取镜像  
docker pull mysql:8  
查看是否拉取成功  
docker images  
启动容器  
```
docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
-v /opt/docker_v/mysql/conf:/etc/mysql/conf.d：将主机/opt/docker_v/mysql/conf目录挂载到容器的/etc/mysql/conf.d
-e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码
-d: 后台运行容器，并返回容器ID
imageID: mysql镜像ID
```
或者在容器创建一个网络来启动容器  
```bash
docker network create mysql-network
docker run -d --name mysql-master --network=mysql-network -e MYSQL_ROOT_PASSWORD=master_password mysql:latest
```

查看是否运行成功  
docker ps  
## docker实现Mysql的主从复制  
新建一个主服务器  
```
docker run -d -p 3306:3306 --name mysql_master -e MYSQL_ROOT_PASSWORD=123456 -v /mydata/mysql-master/log:/var/log/mysql -v /mydata/mysql-master/data:/var/lib/mysql -v /mydata/mysql-master/conf:/etc/mysql  mysql:5.7
```
修改my.cnf  
在/mydata/mysql-master/conf 文件夹下新建一个my.cnf文件，然后下面的内容粘贴进去  
```
[mysqld]
## 设置serverid,同一个局域网内要唯一
server_id=101
##指定不需要同步的数据库名称
binlog-ignore-db=mysql
##开启二进制日志功能
log-bin=mall-mysql-bin
##设置二进制日志使用内存大小(事务)
binlog_cache_size=1M
##设置使用的二进制日志格式
binlog_format=mixed
##二进制日志过期清理时间
expire_logs_days=7
##跳过主从复制中所有错误或指定类型的错误，避免slave端复制中断
###1062主键重复，1032主重数据不一致
slave_skip_errors=1062
##允许远程登录
bind-address=0.0.0.0
```bash
重启主服务器
```bash
docker restart mysql_master
```
进入主服务器  
进入容器内  
```bash
docker exec -it mysql_master /bin/bash
```
登录到mysql  

```bash
mysql -uroot -p123456
```

master容器实例内创建数据同步用户  

```
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```
在主数据库查看主从同步状态  
```bash
show master status \G;
```
这里的参数很关键，等会在从数据库最后配置的时候需要从这里获取对应的值  
新建从服务器  
```bash
docker run -d -p 3307:3306 --name mysql_slave -e MYSQL_ROOT_PASSWORD=123456 -v /mydata/mysql-slave/log:/var/log/mysql -v /mydata/mysql-slave/data:/var/lib/mysql -v /mydata/mysql-slave/conf:/etc/mysql  mysql:5.7
```

修改my.cnf  
```
[mysqld]
server_id=102
##指定不需要同步的数据库名称
binlog-ignore-db=mysql
##开启二进制日志功能
log-bin=mall-mysql-slave1-bin
##设置二进制日志使用内存大小(事务)
binlog_cache_size=1M
##设置使用的二进制日志格式
binlog_format=mixed
##二进制日志过期清理时间
expire_logs_days=7
##跳过主从复制中所有错误或指定类型的错误，避免slave端复制中断
###1062主键重复，1032主重数据不一致
slave_skip_errors=1062
##配置中继日志
relay_log=mall-mysql-relay-bin
##表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
##slave设置为只读
read_only=1
##允许远程登录
bind-address=0.0.0.0
```
重启从服务器  
```bash
docker restart mysql_slave
```
进入从服务器  
进入容器内  
```bash
docker exec -it mysql_slave /bin/bash
```
登录MySQL  
```bash
mysql -uroot -p123456
```
在从数据库中配置主从配置  
```bash

change master to master_host='192.168.40.128', master_user='slave', master_password='123456', master_port=3306;
```
刚开始先这样写就好，也可以指定日志：  

```bash
change master to master_host='192.168.40.128', master_user='slave', master_password='123456', master_port=3306, master_log_file='mall-mysql-bin.000001', master_log_pos=928, master_connect_retry=30;
```

master_host：主数据库的IP地址；  
master_port：主数据库的运行端口；  
master_user：在主数据库创建的用于同步数据的用户账号；  
master_password：在主数据库创建的用于同步数据的用户密码；  
master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；  
master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；  
master_connect_retry：连接失败重试的时间间隔，单位为秒。  
这里的参数由上面 show master status; 得到  
在从数据库查看主从同步状态  
```bash
show slave status \G;
```
关注   
Slave_IO_Runing与Slave_SQL_Running是不是Yes  
开启主从同步  
```bash
start slave；
```

测试：  
在主数据库中新建一个数据库叫做mydata  

这样就完成了，注意几个坑  

主机可能需要刷新权限  

```bash
FLUSH PRIVILEGES;
```
这个参数需要设置为OFF  

```bash
 show variables like 'skip_networking ';
```

防火墙设置：  
```
# 对于 ufw
sudo ufw allow 3306
# 对于 firewalld
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```

# 主从同步延迟怎么处理？  
主从同步延迟的原因  
一个服务器开放Ｎ个链接给客户端来连接的，这样有会有大并发的更新操作, 但是从服务器的里面读取 binlog 的线程仅有一个，当某个 SQL 在从服务器上执行的时间稍长 或者由于某个 SQL 要进行锁表就会导致，主服务器的 SQL 大量积压，未被同步到从服务器里。这就导致了主从不一致， 也就是主从延迟。  
  主从同步延迟的解决办法  
解决主从复制延迟有几种常见的方法:  

1、写操作后的读操作指定发给数据库主服务器  
例如，注册账号完成后，登录时读取账号的读操作也发给数据库主服务器。这种方式和业务强绑定，对业务的侵入和影响较大，如果哪个新来的程序员不知道这样写代码，就会导致一个 bug。  

2、读从机失败后再读一次主机  
这就是通常所说的 "二次读取" ，二次读取和业务无绑定，只需要对底层数据库访问的 API 进行封装即可，实现代价较小，不足之处在于如果有很多二次读取，将大大增加主机的读操作压力。例如，黑客暴力破解账号，会导致大量的二次读取操作，主机可能顶不住读操作的压力从而崩溃。

3、关键业务读写操作全部指向主机，非关键业务采用读写分离  
例如，对于一个用户管理系统来说，注册 + 登录的业务读写操作全部访问主机，用户的介绍、爰好、等级等业务，可以采用读写分离，因为即使用户改了自己的自我介绍，在查询时却看到了自我介绍还是旧的，业务影响与不能登录相比就小很多，还可以忍受。  
查看binlog  
```
查看binlog全部文件
show binary logs;#查看binlog是否开启NO为开启
show variables like 'log_bin%';#详细信息
show variables like 'binlog%';#查看binlog日志
show binlog events in'mysql-bin.000007';#或者使用mysqlbinlog，如果报错使用--no-defaults（使用全路径）[root@localhost ~]# /usr/local/mysql/bin/mysqlbinlog --no-defaults /usr/local/mysql/data/mysql-bin.000019
```
https://blog.csdn.net/javaboyweng/article/details/125441670







