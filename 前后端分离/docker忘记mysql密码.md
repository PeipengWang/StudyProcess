# docker忘记mysql密码

解决方法
1.首先启动mysql容器
docker exec -it mysql /bin/bash
2.编辑mysql配置文件
这里非常重要，千万不要去那什么写成了这个 vi /etc/mysql/my.cnf
vi /etc/mysql/conf.d/docker.cnf
3.打开之后是这样的，skip-grant-tables 需要自己加上去 这句话的意思是进入忘记密码模式
[mysqld]
skip-host-cache
skip-name-resolve
skip-grant-tables
4.修改完之后，使用exit命令退出容器
5.之后重启下mysql容器
docker restart mysql
6.再次进入mysql容器
docker exec -it mysql /bin/bash
7.连接mysql
mysql -u root -p
8.之后你会发现还是需要你输入密码，这个时候密码可以随便输入了，不管输入什么都可以登录了
9.如果需要重置密码的话
mysql> use mysql;
mysql> update user set password=password("这里输入想要修改的密码") where user='root';
mysql> flush privileges;
