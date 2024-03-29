# 安装mysql
## 检查MariaDB
因为这个会和MySQL有冲突，所以先检查一下是否有安装
#查看mariadb
rpm -qa|grep mariadb
#卸载mariadb，mariadb-libs-5.5.44-2.el7.centos.x86_64是上一步查看显示出来的
rpm -e --nodeps mariadb-libs-5.5.44-2.el7.centos.x86_64
#删除etc目录下的my.cnf
rm /etc/my.cnf
## 检查是否已经存在MySQL
#查看mariadb
rpm -qa | grep mysql
#卸载mysql,mysql-libs-5.1.73-5.el6_6.x86_64是上一步查看显示的结果
rpm -e --nodeps mysql-libs-5.1.73-5.el6_6.x86_64
## 官网下载安装包
官网地址https://downloads.mysql.com/archives/community/，下载对应版本的安装包，如果不知道版本，输入如下命令即可查看
## 解压
进入/usr/local下，输入如下命令解压
cd /usr/local/
tar -xvf /home/mysql-8.0.28-linux-glibc2.12-x86_64.tar.xz
解压完成之后，给解压好的文件夹改个名字
mv mysql-8.0.28-linux-glibc2.12-x86_64 mysql
##  创建data目录
mkdir /usr/local/mysql/data
## 创建MySQL用户组和用户
#进入mysql目录
cd /usr/local/mysql
#添加用户组
groupadd mysql
#添加用户
useradd -g mysql mysql
## 更改权限
为了方便，直接给mysql文件夹更改权限
chown -R mysql.mysql /usr/local/mysql
## 配置环境
配置my.cnf文件，如果没有则使用如下命令创建
touch /etc/my.cnf
如果有，直接编辑
```
[mysql]
#设置mysql客户端默认字符集
default-character-set=utf8
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
socket=/usr/local/mysql/mysql.sock
symbolic-links=0
[client]
port=3306
socket=/usr/local/mysql/mysql.sock
!includedir /etc/my.cnf.d
```
## 初始化
进入mysql的bin目录（第二条命令不要忘记修改自己的mysql安装目录）
cd /usr/local/mysql/bin/
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/ --user=mysql --initialize
此时，如果linux环境缺少libaio.so.1，会报错./mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
安装对应的缺少文件后，再执行刚才的命令
yum install -y libaio.so.1
#如果执行了上边的命令不能解决，那么在执行下边的即可
yum install -y libaio
## 加入系统服务
同样，再次提醒，不要忘记修改自己的mysql安装目录
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
chkconfig --add mysql
## 启动MySQL，修改密码
service mysql start
#进入mysql
./mysql -u root -p
修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
#刷新权限
flush privileges;
## 配置远程访问
使用刚才设置的密码进行登录，登录成功后输入下面的命令
use mysql;
select user,host,plugin,authentication_string from user;
#mysql8认证方式改了,mysql_native_password这个才能远程连接mysql
alter user 'root'@'%' identified with mysql_native_password by 'root';
flush privileges;
# docker安装mysql
docker run \
--name mysql \
-d \
-p 3306:3306 \
--restart unless-stopped \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=db0$ZTE \
mysql:5.6

https://www.cnblogs.com/whoyoung/p/10988136.html