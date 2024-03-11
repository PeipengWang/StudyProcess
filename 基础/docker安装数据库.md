# docker安装数据库
docker run -d -p 3306:3306 --name mysql_master -e MYSQL_ROOT_PASSWORD=123456 -v /mydata/mysql-master/log:/var/log/mysql -v /mydata/mysql-master/data:/var/lib/mysql -v /mydata/mysql-master/conf:/etc/mysql  mysql:5.7

docker run -d -p 3307:3306 --name mysql_slave -e MYSQL_ROOT_PASSWORD=123456 -v /mydata/mysql-slave/log:/var/log/mysql -v /mydata/mysql-slave/data:/var/lib/mysql -v /mydata/mysql-slave/conf:/etc/mysql  mysql:5.7

change master to master_host='121.4.15.75', master_user='slave', master_password='123456', master_port=3306, master_log_file='mall-mysql-bin.000001', master_log_pos=928, master_connect_retry=30;
CHANGE MASTER TO MASTER_HOST='172.17.0.2
', MASTER_USER='root', MASTER_PASSWORD='123456', MASTER_PORT=3306, MASTER_LOG_FILE='mall-mysql-bin.000001', MASTER_LOG_POS=617, MASTER_CONNECT_RETRY=30;
STOP SLAVE IO_THREAD FOR CHANNEL 'slave';



change master to master_host='121.4.15.75', master_user='slave', master_password='123456', master_port=3306;