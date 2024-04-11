# 启动oracle
启动数据库实例，分为两步：第一步，启动监听；第二步，启动数据库实例。
一、如何启动数据库实例
 1.进入到sqlplus启动实例
--“切换到oracle用户”
su - oracle
--“打开监听”

lsnrctl start 
  --“进入到sqlplus”
sqlplus /nolog 
--“连接到sysdba”
conn /as sysdba
 或者上述两部合并为
 sqlplus / as sysdba
--“启动数据库实例”
startup
以普通用户登录
sqlplus user/'Aa#12345'
# 关闭
关闭服务
停止 Oracle 监听服务
$ lsnrctl stop
停止 Oracle 服务
$ sqlplus /nolog
SQL > conn / as sysdba
SQL > shutdown immediate
SQL > exit
