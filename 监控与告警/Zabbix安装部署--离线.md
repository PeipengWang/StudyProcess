# 关闭SELinux

setenforce 0

vim /etc/selinux/config

更改SELINUX=disabled

# 关闭防火墙

firewall-cmd --state

systemctl stop firewalld.service

systemctl disable firewalld.service

 **建议改进**：
考虑以下方法提高安全性：

- 将 SELinux 设置为 

  ```
  permissive
  ```

   模式，而不是完全禁用：

  ```
  bash复制代码setenforce 0
  sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
  ```

- 配置防火墙允许必要端口（如 Zabbix 使用的 10050、10051 等）：

  ```
  bash复制代码firewall-cmd --permanent --add-port=10050-10051/tcp
  firewall-cmd --reload
  ```

# 关闭yum

配置yum

vim /etc/yum.repos.d/zabbix.repo

更改enabled=0

# 服务端安装mysql 

groupadd mysql

useradd -r -g mysql -s /bin/false mysql

进入解压后的mysql目录

bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

查看/etc/my.cnf，如果没有可手动配置

安装服务：cp support-files/mysql.server /etc/init.d/mysqld

systemctl start mysqld

systemctl enable mysqld

mysql -uroot -p ：输入初始化的密码，登录mysql



bug1:my.cnf 是空的；

解决方式：手写；

bug2：the server quit without updating PID file ...

解决方式:查看错误日志，发现是data dictionary 初始化失败，删除mysql/data文件，更新权限，重新初始化mysql，再restart服务

bug3: your password is expired

解决方式：my.cnf中增加skip-grant-tables,重启服务，用初始密码登录，跟更mysql库中信息；use mysql;update user set password_expire="N" where user="root";flush privileges;quit;注释skip-grant-tables,重启服务；

修改密码

# 服务端Zabbix数据库

进入mysql命令行，创建zabbix数据库

CREATE DATABASE zabbix character set utf8 collate utf8_bin;

GRANT all ON zabbix.* TO ‘zabbix’@’localhost’ IDENTIFIED BY ‘zabbix’@’localhost’;

flush privileges;

quit;

表结构

zcat /../zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

输入密码

查看表结构

# 服务端安装zabbix

```
mkdir -p /usr/local/zabbix-server-repo
cp /path/to/zabbix-server/*.rpm /usr/local/zabbix-server-repo/
```

复制server文件下包到

```
cd /usr/local/zabbix-server-repo/
yum clean all;
yum -y localinstall ./*.rpm --skip-broken
```

或者只安装zabbix相关rpm

```
优化安装命令
yum -y localinstall zabbix-server-mysql-5.0.18-1.el8.x86_64.rpm \
zabbix-web-mysql-5.0.18-1.el8.noarch.rpm \
zabbix-apache-conf-5.0.18-1.el8.noarch.rpm \
zabbix-agent-5.0.18-1.el8.x86_64.rpm \
zabbix-web-5.0.18-1.el8.noarch.rpm \
zabbix-web-deps-5.0.18-1.el8.x86_64.rpm \
php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm \
php-mysqlnd-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm \
php-bcmath-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm \
php-gd-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm \
php-json-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm \
php-mbstring-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm \
php-xml-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm \
php-pdo-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64.rpm
```

# 配置 Zabbix 和相关依赖服务

这一步主要是配置 Zabbix 和相关依赖服务（如 Apache 和 PHP）的配置文件，以确保它们能够正确运行并支持 Zabbix 的前端和后端服务。以下是详细操作步骤：

------

### **1. 修改 Apache 配置**

编辑 Apache 的主配置文件：

```bash
vim /etc/httpd/conf/httpd.conf
```

#### 修改内容：

- **添加 `ServerName`** 找到或添加以下行，指定服务器名称：

  ```apache
  ServerName www.zabbixyk.com
  ```

- **设置默认索引文件** 找到 `DirectoryIndex` 指令行，确认或修改为以下内容：

  ```apache
  DirectoryIndex index.htm index.php
  ```

- 保存并退出（`:wq`）。

------

### **2. 修改 Zabbix 服务端配置**

编辑 Zabbix 服务端的主配置文件：

```bash
vim /etc/zabbix/zabbix_server.conf
```

#### 配置内容：

- **设置数据库用户和密码** 找到以下行（如果不存在则添加）并设置数据库的用户名和密码：

  ```conf
  DBHost=localhost
  DBName=zabbix
  DBUser=zabbix
  DBPassword=你的数据库密码
  ```

- 保存并退出。

------

### **3. 修改 PHP 配置**

编辑 PHP 的配置文件：

```bash
vim /etc/php.ini
```

#### 修改内容：

- **启用 `always_populate_raw_post_data`** 找到或添加以下行：

  ```ini
  always_populate_raw_post_data = -1
  ```

- **设置时区** 找到或添加以下行，确保 PHP 以正确的时区运行：

  ```ini
  date.timezone = Asia/Shanghai
  ```

- 保存并退出。

------

### **4. 修改 Zabbix 前端的 Apache 配置**

编辑 Zabbix 前端的 Apache 配置文件：

```bash
vim /etc/httpd/conf.d/zabbix.conf
```

#### 修改内容：

- **设置 PHP 时区** 确保文件中包含以下行：

  ```apache
  php_value date.timezone Asia/Shanghai
  ```

- 保存并退出。

------

### **5. 启动和启用服务**

运行以下命令以启动 Zabbix 服务端、Zabbix Agent 和 Apache 服务：

```bash
systemctl start zabbix-server zabbix-agent httpd
```

确保服务在系统启动时自动启动：

```bash
systemctl enable zabbix-server zabbix-agent httpd
```

------

### **6. 验证服务运行状态**

运行以下命令，确保服务启动成功：

```bash
systemctl status zabbix-server zabbix-agent httpd
```

------

### **7. 测试访问**

在浏览器中访问 Zabbix 前端，URL 为：

```url
http://你的服务器IP或域名/zabbix
```

使用默认账号 `Admin` 和密码 `zabbix` 登录。如果页面无法访问或报错，请检查 Apache 和 PHP 配置是否正确，或查看错误日志：

```bash
# Apache 日志
cat /var/log/httpd/error_log

# Zabbix 日志
cat /var/log/zabbix/zabbix_server.log
```

完成以上步骤后，Zabbix 服务端和前端应该可以正常运行。

# 客户端安装agent

关闭SELinux

关闭yum

复制agent文件目录

yum -y localinstall ./* --skip-broken

更改配置文件

vim /etc/zabbix/zabbix_agentd.conf

配置logfilesize=100,server=IP activeServer=IP Hostname=Zabbix server （相同）

systemctl start zabbix-agent 

systemctl enable zabbix-agent 

1. 更改配置文件

vim /etc/httpd/conf/httpd.conf

添加 ServerName [www.zabbixyk.com](http://www.zabbixyk.com)

directoryIndex index.htm index.php

 

vim /etc/zabbix/zabbix_server.conf

配置DB用户和密码

 

vim /etc/php.ini

 

always_populate_raw_post_data = -1

时区Asia/Shanghai

 

vim /etc/httpd/conf.d/zabbix.conf

php_value date.timezone Asia/Shanghai

![img](file:///C:\Users\wangp\AppData\Local\Temp\ksohtml32028\wps13.jpg) 

启动服务

systemctl start zabbix-server zabbix-agent httpd

systemctl enable zabbix-server zabbix-agent httpd

 

2. 客户端安装agent

关闭SELinux

关闭yum

复制agent文件目录

 

yum -y localinstall ./* --skip-broken

 

更改配置文件

vim /etc/zabbix/zabbix_agentd.conf

配置logfilesize=100,server=IP activeServer=IP Hostname=Zabbix server （相同）

 

systemctl start zabbix-agent 

systemctl enable zabbix-agent 

 

3. Zabbix前端配置

 

mysql root> select user, host,plugin from mysql.user;

mysql zabbix>alter user 'zabbix'@'%' identified with mysql_native_password by '123456';

谷歌浏览器进入www.172.24.1.1/zabbix

账号:Admin

密码:

配置→主机→选择已有的→完全克隆→更改IP→更新

 

进入某主机，创建应用集MonitorShow，进入应用集，点击右侧监控项，创建监控项；输入名称，键值（proc.num[,,,OriginalDataProcessor_3-1.0-SNAPSHOT.jar]等），选择应用集，确认；创建触发器，输入名称，添加表达式，选择监控项，按需更改语句，添加即可；

 

进入应用集，选择Storage的监控项，创建两个监控项，键值分别为vfs.fs.size[/pt_data,free]和vfs.fs.size[/pt_data,total]，其中，/pt_data为要监控的共享存储。

如果分为近线和在线，另外创建相应监控项即可。

![img](file:///C:\Users\wangp\AppData\Local\Temp\ksohtml32028\wps14.jpg) 

 

在模板中添加

CPU监控项: 80%触发器

![img](file:///C:\Users\wangp\AppData\Local\Temp\ksohtml32028\wps15.jpg) 

![img](file:///C:\Users\wangp\AppData\Local\Temp\ksohtml32028\wps16.jpg) 

 

 

 

 

Memory监控项：内存使用百分比

可计算类型，浮点数

键值如下：100*(last("vm.memory.size[total]")-last("vm.memory.size[available]"))/last("vm.memory.size[total]")

 

应用集 Filesystem / 

监控项

![img](file:///C:\Users\wangp\AppData\Local\Temp\ksohtml32028\wps17.jpg) 

 

# 问题：缺少libevent-2.1.so.6

系统需要安装libevent

离线方式：

提前下载

[libevent-2.1.8-stable.tar.gz](https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz) 

官方网址：

https://libevent.org/

## 源码方式安装步骤

一、解压缩源码包。

```
tar -xzvf libevent-2.1.8-stable.tar.gz
```

 二、进入libevent-2.1.8-stable目录，执行 configure 配置脚本

 1、查看configure 脚本的使用帮助及其选项，可以执行命令：./configure --help 查看。

如果直接执行：./configure，那么默认安装路径是/usr/local，对应的头文件、可执行文件和库文件分别对应的目录是：'/usr/local/include'、'/usr/local/bin'，'/usr/local/lib'。

2、我本人设置了默认安装路径，执行命令如下：

```
./configure
```

 3、第2步执行成功后，会生成Makefile文件，然后使用make命令进行源码编译。

```
make
```

 4、编译成功后，执行安装命令。

```
make  install
```

5、进行软连接

```
ln -s /usr/local/lib/libevent-2.1.so.6 /usr/lib/libevent-2.1.so.6
```

或者将 /usr/local/lib目录加入环境变量

编辑.bashrc文件，添加

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

已经有库了还是不行可以使用如下方法：

1. 首先打开/etc/ld.so.conf文件
2. 加入动态库文件所在的目录：执行vi /etc/ld.so.conf，在"include ld.so.conf.d/*.conf"下方增加"/usr/local/lib"。这里这个文件为只读文件，可以使用以下操作对其改写：

```c
sudo vim ld.so.conf
1
```

修改完成后使用wq！退出即可。

1. 保存后，在命令行终端执行：/sbin/ldconfig -v；其作用是将文件/etc/ld.so.conf列出的路径下的库文件缓存到/etc/ld.so.cache以供使用，因此当安装完一些库文件，或者修改/etc/ld.so.conf增加了库的新搜索路径，需要运行一下ldconfig，使所有的库文件都被缓存到文件/etc/ld.so.cache中，如果没做，可能会找不到刚安装的库。

经过以上三个步骤，"error while loading shared libraries"的问题通常情况下就可以解决了

## rpm包安装步骤

------

#### **1. 检查系统是否已安装 `libevent`**

在终端运行以下命令，检查是否已经安装：

```bash
rpm -qa | grep libevent
```

如果结果中包含 `libevent` 的版本号，并且已满足需求，则无需重复安装。

#### **2. 下载 `libevent` 的 RPM 包**

根据操作系统版本（如 CentOS/RHEL 7 或 8）和架构（如 x86_64），从以下网站下载适合的 RPM 包：

- **CentOS 官方仓库**（包含常用 RPM 包）：https://vault.centos.org/
- **RPMFind 仓库**：https://rpmfind.net/

搜索 `libevent`，选择版本 `2.1.x`，并下载相应的 RPM 包。例如：

```bash
wget http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/libevent-2.1.8-5.el8.x86_64.rpm
```

------

#### **3. 手动安装 RPM 包**

在终端中运行以下命令：

```bash
yum install -y ./libevent-2.1.8-5.el8.x86_64.rpm
```

------

#### **4. 验证安装**

安装完成后，验证 `libevent` 是否成功安装，并检查是否包含 `libevent-2.1.so.6`：

```bash
ls -l /usr/lib64/libevent-2.1.so.6
```

如果文件存在，说明安装成功。

------

#### **5. 处理依赖问题（如果有）**

如果安装时提示依赖问题，可能需要下载其他依赖包（如 `libevent-devel` 或相关依赖）。可以运行以下命令解决：

```bash
yum deplist ./libevent-2.1.8-5.el8.x86_64.rpm
```

然后根据提示逐一下载依赖包，并安装。

------

#### **6. 确保链接库被加载**

确保 `libevent-2.1.so.6` 被正确加载：

1. 查看当前动态链接库路径：

   ```bash
   echo $LD_LIBRARY_PATH
   ```

2. 如果 

   ```
   /usr/lib64
   ```

    未包含在环境变量中，可通过以下命令添加：

   ```bash
   export LD_LIBRARY_PATH=/usr/lib64:$LD_LIBRARY_PATH
   ```

3. 更新链接库缓存：

   ```bash
   ldconfig
   ```

------

### **总结**

1. 下载适配的 `libevent` RPM 包。
2. 使用 `yum install` 安装。
3. 确认文件安装路径和动态链接是否正确加载。

若仍有问题，请提供详细的错误信息以进一步分析。