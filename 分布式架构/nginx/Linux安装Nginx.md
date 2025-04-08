# ngixn安装
## 安装依赖包
···
yum install gcc
yum install pcre-devel
yum install zlib zlib-devel
yum install openssl openssl-devel
···
## 安装nginx
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

···
[root@vm_test_single bin]# ps -ef | grep nginx
root     28486     1  0 09:29 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   28487 28486  0 09:29 ?        00:00:00 nginx: worker process
···

## 配置软链接
#创建软链接  

ln -s /usr/local/nginx/sbin/nginx  /usr/bin/  

## 配置防火墙  
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



## nginx “403 Forbidden” 错误的原因及解决办法  
权限配置不正确  

这个是nginx出现403 forbidden最常见的原因。  

为了保证文件能正确执行，nginx既需要文件的读权限,又需要文件所有父目录的可执行权限。  

例如，当访问/usr/local/nginx/html/image.jpg时，nginx既需要image.jpg文件的可读权限，也需要/, /usr,/usr/local,/usr/local/nginx,/usr/local/nginx/html的可以执行权限。  

解决办法:设置所有父目录为755权限，设置文件为644权限可以避免权限不正确。  

chmod -R +x /some/directory  

1.注意nginx的配置文件 user nobody改成user root  

## 发现报以下错误：  
nginx: [error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)  

解决办法，执行以下语法即可修复：  

/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf  
