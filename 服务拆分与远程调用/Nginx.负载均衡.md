# Nginx
## 配置
使用默认配置，在nginx根目录下执行
./configure
make
make install
查找安装路径： whereis nginx
Nginx常用命令
cd /usr/local/nginx/sbin/
./nginx  启动
./nginx -s stop  停止
./nginx -s quit  安全退出
./nginx -s reload  重新加载配置文件
ps aux|grep nginx  查看nginx进程
启动成功访问 服务器ip:80
注意：如何连接不上，检查阿里云安全组是否开放端口，或者服务器防火墙是否开放端口！
相关命令：
 开启
service firewalld start
 重启
service firewalld restart
 关闭
service firewalld stop
查看防火墙规则
firewall-cmd --list-all
查询端口是否开放
firewall-cmd --query-port=8080/tcp
开放80端口
firewall-cmd --permanent --add-port=80/tcp
移除端口
firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
演示
upstream lb{
    server 127.0.0.1:8080 weight=1;
    server 127.0.0.1:8081 weight=1;
}
location / {
    proxy_pass http://lb;
}

正向代理代理的是客户端
反向代理代理的是客户端