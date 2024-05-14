# Nginx--实战 取代Apache的高性能web服务器

## Nginx简介

Nginx是一款高性能的HTTP和反向代理服务器，它能支持高达5000个高并发连接数的响应，而内存、CPU等消耗确非常低，运行非常稳定
配置文件简单
支持Rewrite重写规则，能够根据域名、URL不同，将HTTP请求分到不同的后端服务群组。

## Nginx安装

### 安装nginx

下载nginx的tar包
登录http://nginx.org/en/download.html，下载nginx的Stable version版本，并解压
#执行configure命令
./configure  --with-http_ssl_module
#执行make命令
make
#执行make install命令
make install
#Nginx常用命令
#测试配置文件安装路径下的
/usr/local/nginx/sbin/nginx -t
#启动命令安装路径下的
/usr/local/nginx/sbin/nginx
#停止命令安装路径下的
/usr/local/nginx/sbin/nginx -s stop
#或者 : 
/usr/local/nginx/sbin/nginx -s quit
#重启命令
安装路径下的/usr/local/nginx/sbin/nginx -s reload
#查看进程命令
[root@vm_test_single bin]# ps -ef | grep nginx
[root@vm_test_single bin]# ps -ef | grep nginx
root     28486     1  0 09:29 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   28487 28486  0 09:29 ?        00:00:00 nginx: worker process

### 配置软链接

#创建软链接
ln -s /usr/local/nginx/sbin/nginx  /usr/bin/

### 配置防火墙

#打开防火墙文件
vim /etc/sysconfig/iptables
#新增行  开放80端口
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT//保存退
#重启防火墙
service iptables restart
#Nginx虚拟域名配置及测试验证
#编辑nginx.conf
vim /usr/local/nginx/conf/nginx.conf
#增加行 
include vhost/*.conf

### nginx “403 Forbidden” 错误的原因及解决办法

权限配置不正确
这个是nginx出现403 forbidden最常见的原因。
为了保证文件能正确执行，nginx既需要文件的读权限,又需要文件所有父目录的可执行权限。
例如，当访问/usr/local/nginx/html/image.jpg时，nginx既需要image.jpg文件的可读权限，也需要/, /usr,/usr/local,/usr/local/nginx,/usr/local/nginx/html的可以执行权限。
解决办法:设置所有父目录为755权限，设置文件为644权限可以避免权限不正确。
chmod -R +x /some/directory
1.注意nginx的配置文件 user nobody改成user root

### 发现报以下错误：

nginx: [error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
解决办法，执行以下语法即可修复：
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

## 基本指令

指定配置文件启动
nginx -c nginx.conf
直接启动
nginx
重启
nginx -s reload
关闭
nginx -s stop

## 配置文件

### 全局配置

user  root;
使用的用户和组
worker_processes  1;
指工作衍生的进程数（一般等于CPU的总核数或总核数的两倍，例如四个CPU，则设置为8）
error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;
设置日志存放的路径和级别
pid        logs/nginx.pid;
指定pid存放的路径

events {
    use  epoll 使用网络I/O模型，Linux系统推荐采用epoll模型，FreeBSD系统推荐采用kqueue模型
    worker_connections  1024;   允许连接的数目
}

### 虚拟主机配置

Nginx可以配置多种类型的虚拟主机：基于IP的虚拟主机，基于域名的虚拟主机，基于端口的虚拟主机

```
http {
    include       mime.types;
    default_type  application/octet-stream; 使用的字符集
    log_format  main  '$remote_addr -        $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"';
    //日志访问格式：$remote_addr将反向代理服务器的地址 
    access_log  logs/access.log  main;
    //指定日志存放路径，如果不想记录日志可以设置为 access_log off;
    sendfile        on;
    tcp_nopush     on;

    keepalive_timeout  0;
    keepalive_timeout  65;
    gzip  on;

    server {
        listen       10.235.5.176:3000; // 监听的IP和端口,不写IP则监听server_name所有IP的端口
        server_name  localhost; //主机名称
        #charset koi8-r;
        access_log  logs/host.access.log  main; //访问日志存放路径
        location / {
            root   html;  //HTML网页文件存放路径
            index  index.html index.htm; //默认首页
        }
}
```

### 设置浏览器本地缓存

语法：expires [time|epoch|max|off]
默认值：expire off
作用域：http server location
用途：使用本指令可以控制HTTP应答中的Expires和Cache-Control的Head头信息（起到页面缓存作用）
例如：expires 30d 30天
          expires 1h 1小时

