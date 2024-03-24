# Mysql远程连接设置
1、设置允许远程连接
首先打开 mysqld.cnf 配置文件。
```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
找到 bind - address 这一行
默认情况下， bind - address 的值为 127.0.0.1 ，所以只能监听本地连接。我们需要将这个值改为远程连接 ip 可访问，可使用通配符 ip 地址 *， ::， 0.0.0.0 ，当然也可以是单独的固定 ip，这样就仅允许指定 ip 连接，更加安全。
在某些 MySQL 版本的配置文件中，没有 bind - address 这一行，这种情况下，在合适的位置加上就可以了。
然后重启 MySQL 服务，使刚刚编辑的 mysqld.cnf 文件生效：
```
sudo systemctl restart mysql
```
2、设置连接数
方法一：进入MySQL用命令行修改，但是MySQL重启后就失效了，需要重新设置。（不推荐）
　　命令如下：
　　1、show variables like 'max_connections';（查看当前最大连接数）
　　2、set global max_connections=1000;（设置最大连接数为1000，可以再次执行上面的命令查看设置是否成功）
　　方法二：修改MySQL配置文件，永久生效（推荐）
　　1、进入MySQL安装目录，打开my.ini或my.cnf文件；
　　2、查看max_connections=100的配置参数，修改为max_connections=1000；如果没有这个参数，直接添加max_connections=1000即可；
　　3、保存配置文件并退出，重启MySQL服务即可。
1、查看数据库当前连接信息,可以看到连接数据库的进程id，ip,用户名，连接的数据库，连接状态，连接时长等
　　命令：SHOW FULL processlist