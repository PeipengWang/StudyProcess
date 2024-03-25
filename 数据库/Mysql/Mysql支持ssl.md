
# Mysql支持ssl
查看数据库是否支持ssl配置

mysql> show variables like 'have%ssl%';

查看数据库端口号

mysql> show variables like 'port';


查看数据库数据存放路径

mysql> show variables like 'datadir';


通过openssl 制作生成 SSL 证书（有效期99999）

1、生成一个 CA 私钥

[root@itop ~]#openssl genrsa 2048 > ca-key.pem

2、通过 CA 私钥生成数字证书

[root@itop ~]# openssl req -new -x509 -nodes -days 99999 -key ca-key.pem -out ca.pem

3、创建 MySQL 服务器 私钥和请求证书

[root@itop ~]# openssl req -newkey rsa:2048 -days 99999 -nodes -keyout server-key.pem -out server-req.pem

4、将生成的私钥转换为 RSA 私钥文件格式

[root@itop ~]# openssl rsa -in server-key.pem -out server-key.pem

5、用CA 证书来生成一个服务器端的数字证书

[root@itop ~]# openssl x509 -req -in server-req.pem -days 99999 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem

6、创建客户端的 RSA 私钥和数字证书

[root@itop ~]# openssl req -newkey rsa:2048 -days 99999 -nodes -keyout client-key.pem -out client-req.pem

7、将生成的私钥转换为 RSA 私钥文件格式

[root@itop ~]# openssl rsa -in client-key.pem -out client-key.pem

8、用CA 证书来生成一个客户端的数字证书

[root@itop ~]# openssl x509 -req -in client-req.pem -days 99999 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem

据库配置SSL证书
1、复制 CA 证书和服务端SSL文件至MySQL 数据目录

[root@itop ~]# cp ca.pem server-*.pem /data/mysql/data –v

2、修改 MySQL 数据目录的CA 证书和服务端 SSL 文件所属用户与组

[root@itop ~]# chown -v mysql.mysql /data/mysql/data/{ca,server*}.pem

3、修改MYSQL配置文件，添加SSL调用配置【/etc/my.cnf】

vi /etc/my.cnf

4、重启MYSQL服务，并检查数据库SSL是否开启状态

注：have_openssl 与 have_ssl 值都为YES表示ssl开启成功

