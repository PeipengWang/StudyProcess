1、检测是否有安装redis-cli和redis-server;
[root@localhost bin]# whereis redis-cli
redis-cli: /usr/bin/redis-cli

[root@localhost bin]# whereis redis-server
redis-server: /usr/bin/redis-server
说明已经安装好了，如果不知道怎么安装，告诉你一个简单的方法，一步就可以把php、php-redis拓展，redis-server,redis-cli这三个装起来。 

参考链接：http://blog.csdn.net/zhezhebie/article/details/73325663

接着就把redis-server加入快捷键，谁都不想每次敲一长串地址： 
参考链接：http://blog.csdn.net/zhezhebie/article/details/71641326

 

2、Redis的启动
　　（1）前端模式启动

　　　　①直接运行bin/redis-server将以前端模式启动：切换到 /usr/local/redis/bin目录下，然后./redis-server ；

　　　　②前端模式的缺点是启动完成后，不能再进行其他操作；

　　　　③如果要操作必须使用Ctrl+C，同时redis-server程序结束，不建议使用此方法。

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/exam16b2d32ce62b24fa81c851751d89d86e.png)

　　（2）通过指定配置文件启动

　　　　①参考06003_redis在Linux上的安装

　　　　第8步，修改redis.conf，把daemonize no修改成daemonize yes，保存退出；

　　　　②启动时，指定配置文件；

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/exam7a87cec50d4701c94b84b57f7ef8533a.png)　　　

　　　　③Redis默认端口6379，通过当前服务进行查看。

　　　　![](https://raw.githubusercontent.com/PeipengWang/picture/master/exam7a87cec50d4701c94b84b57f7ef8533a.png)

　　（3）后端模式启动

```
redis-server &
```


（4）设置Redis开机自启动
 (a)centos7开机自启动：
https://blog.csdn.net/zhezhebie/article/details/75120189
(b)老版本centos的开机自启动方式
=============老版本的centos操作方式，开始分割线============= 
推荐在生产环境中使用启动脚本方式启动redis服务。启动脚本 redis_init_script 位于位于Redis的 /utils/ 目录下。 
大致浏览下该启动脚本，发现redis习惯性用监听的端口名作为配置文件等命名，我们后面也遵循这个约定。 
redis服务器监听的端口

REDISPORT=6379
服务端所处位置，在make install后默认存放与/usr/local/bin/redis-server，如果未make install则需要修改该路径，下同。

EXEC=/usr/bin/redis-server
客户端位置

CLIEXEC=/usr/bin/redis-cli
Redis的PID文件位置

PIDFILE=/var/run/redis_${REDISPORT}.pid
配置文件位置，需要修改

CONF="/etc/redis/${REDISPORT}.conf"
配置环境 
根据启动脚本要求，将修改好的配置文件以端口为名复制一份到指定目录。需使用root用户。
mkdir /etc/redis
cp redis.conf /etc/redis/6379.conf
将启动脚本复制到/etc/init.d目录下，本例将启动脚本命名为redisd（通常都以d结尾表示是后台自启动服务）。

cp redis_init_script /etc/init.d/redisd
设置为开机自启动 

此处直接配置开启自启动
chkconfig redisd on 
将报错误： service redisd does not support chkconfig 

再设置即可成功。 

设置为开机自启动服务器

chkconfig redisd on
打开服务

service redisd start
关闭服务

service redisd stop

3.检测后台进程是否存在
ps -ef |grep redis
检测6379端口是否在监听

netstat -lntp | grep 6379
使用redis-cli客户端检测连接是否正常

redis-cli
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set key "hello world"
OK
127.0.0.1:6379> get key
"hello world"


4、Redis的使用
　　（1）连接客户端

　　　　①在redis的安装目录下有reds的客户端，即redis-cli（Redis Command Line Interface），它是Redis自带的基于命令行的Redis客户端；

　　　②命令：redis-cli -h ip地址 -p 端口

5、Redis的停止
　　（1）强制结束程序。使用kill -9 进程的pid，强行终止Redis进程可能会导致redis持久化丢失；

　　（2）正确停止redis的方式是向redis发送shutdown命令，方法为：（关闭默认端口）

