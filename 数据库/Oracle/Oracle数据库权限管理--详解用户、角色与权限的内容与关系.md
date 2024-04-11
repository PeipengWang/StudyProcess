# Oracle数据库权限管理

## Oracle数据库相关概念
数据库：存储数据的仓库，主要指存放数据的文件，包括数据文件、控制文件、数据库日志文件
数据名：用于唯一的标识数据库
实例：一系列为当前数据库提供服务的内存和后台进程，内存与服务进程的集合称为一个实例 --》 zxin_smap
实例名/SID/服务名：用于唯一的标识一个实例  --》 zxin
实例名与数据库名可以不一致
Oracle服务
OracleService+SID，数据库启动的基础服务
OracleOraDb10g_home1TNSListner，为客户端提供监听服务，提供连接服务
OracleOraDb10g_home1iSQL*Plus，让客户端可以通过浏览器来对数据库进行操作
## 用户

### 创建用户
先用超级管理员连上：sqlplus
输入用户名及密码：scott/tiger
连接成功后，使用：conn sys/sysdba as sysdba;登入超级管理员后，输入如下命令：
create user zhangsan identified by zhangsan default tablespace users quota 10M on users
### 分配角色和权限给用户
分配权限给新用户zhangsan
grant create session, create table, create view to zhangsan
分配角色给新用户
grant RESOURCE to ZHANGSAN;
### 删改用户
drop user 用户名；
更改用户密码语法
alter user 用户名 identified by 新密码
ALTER USER 用户名 IDENTIFIED BY 新密码;

### Oracle的三种登录方式：
sqlplus / as sysdba；  //登陆sys帐户
sqlplus sys as sysdba；//同上
sqlplus scott/tiger；  //登陆普通用户scott
### Oracle的三种启动方式
启动数据库(实例)之前要先启动监听
--- 启动监听
lsnrctl start
--- 查看监听状态
lsnrctl status
--- 停止监听
lsnrctl stop
第一种启动sqlplus方式：sqlplus /nolog(只是启动sqlplus而不连接数据库，使用nolog参数)
监听启动成功后，启动数据库实例

第二种启动sqlplus方式：数据库实例启动成功(没有关闭且一直在运行)后，可以使用其他普通用户登陆连接到数据库了
如果要在启动sqlplus的同时连接到数据库，则需要输入用户名、密码和连接描述符(数据库的网络服务名)
例如 ： sqlplus array/916437@192.168.209.139/brrby
array是我数据库的一个用户名，916437是array用户的登陆密码，192.168.209.139是这个数据库的所在服务器的ip地址，brrby是数据库的网络服务名
第三种启动sqlplus方式：如果是在本机，连接本地的数据库
sqlplus array/916437@brrby
省略IP即可
## 权限分配
Oracle中几乎所有的操作（创建用户、创建表、删除表等等一些增删改查在没有获得权限之前是不允许执行的）在没有获得权限之前都不能够执行,所以有必要先了解一下Oracle中权限的分配。
角色是权限的集合，首先要熟悉角色的概念
### 用户的角色
每个Oracle用户都有一个名字和口令,并拥有一些由其创建的表、视图和其他资源。Oracle角色（role）就是一组权限（privilege） (或者是每个用户根据其状态和条件所需的访问类型)。用户可以给角色授予或赋予指定的权限，然后将角色赋给相应的用户。一个用户也可以直接给其他用户授 权。
3种标准角色：
1. CONNECT Role(连接角色)
临时用户，特别是那些不需要建表的用户，通常只赋予他们CONNECTrole。CONNECT是使用Oracle的简单权限，这种权限只有在对其他用户 的表有访问权时，包括select、insert、update和delete等，才会变得有意义。拥有CONNECT role的用户还能够创建表、视图、序列（sequence）、簇（cluster）、同义词（synonym ）、会话（session）和与其他数据库的链（link）。
2. RESOURCE Role(资源角色)
更可靠和正式的数据库用户可以授予RESOURCE role。RESOURCE提供给用户另外的权限以创建他们自己的表、序列、过程（procedure）、触发器（trigger）、索引（index）和簇（cluster）。
3. DBA Role(数据库管理员角色)
DBA role拥有所有的系统权限----包括无限制的空间限额和给其他用户授予各种权限的能力。SYSTEM由DBA用户拥有。下面介绍一些DBA经常使用的典型权限。
创建角色
除了前面讲到的三种系统角色—-CONNECT、RESOURCE和DBA，用户还可以在Oracle创建自己的role。用户创建的role可以由 表或系统权限或两者的组合构成。为了创建role，用户必须具有CREATE ROLE系统权限。下面给出一个create role命令的实例：
```
create role STUDENT;
```
这条命令创建了一个名为STUDENT的role。
一旦创建了一个role，用户就可以给他授权。给role授权的grant命令的语法与对对用户的语法相同。在给role授权时，在grant命令的to子句中要使用role的名称，如下所示：
```
grant 权限 to STUDENT;
```
删除角色
```
   drop role STUDENT;
```
移除用户角色但不删除
```
   w  STUDENT;
```
查看某个用户拥有的角色
--- 用户名必须大写
select * from dba_role_privs where grantee='ZXIN_SMAP';

修改角色密码
使用 PASSW [ ORD ] 命令可以修改用户口令。任何用户都可以使用该命令修改自身的口令，但是如果要修改其他用户的口令，则必须以DBA身份登录。在SQL*Plus中可以使用该命令取代 ALTER  USER 语句修改用户口令
### 权限
之前已经提到角色与权限的关系：一个角色是多个权限的集合
在系统预定义有三个预定角色
connect 连接
resource 访问资源权限，访问表、序列，不包括create session 
dba 拥有所有权限
还可以自定义角色：create role 角色名
给角色添加权限 ：grant 权限列表|角色列表 to 角色名
其中权限可以分为两类:
数据库系统权限（Database System Privilege）允许用户执行特定的命令集。例如，CREATE TABLE权限允许用户创建表，GRANT ANY PRIVILEGE 权限允许用户授予任何系统权限。
数据库对象权限（Database Object Privilege）使得用户能够对各个对象进行某些操作。例如DELETE权限允许用户删除表或视图的行，SELECT权限允许用户通过select从 表、视图、序列（sequences）或快照 （snapshots）中查询信息。

常见的系统权限
   CREATE SESSION                     创建会话
   CREATE SEQUENCE                    创建序列
   CREATE SYNONYM                     创建同名对象
   CREATE TABLE                       在用户模式中创建表
   CREATE ANY TABLE                   在任何模式中创建表
   DROP TABLE                         在用户模式中删除表
   DROP ANY TABLE                     在任何模式中删除表
   CREATE PROCEDURE                   创建存储过程
   EXECUTE ANY PROCEDURE              执行任何模式的存储过程
   CREATE USER                        创建用户
   DROP USER                          删除用户
授予用户系统权限：GRANT privilege [, privilege...] TO user [, user| role, PUBLIC...]
grant 权限 on 对象名 to username[with grant option 同时获得权限分配权];
查看系统权限
dba_sys_privs  --针对所有用户被授予的系统权限
user_sys_privs --针对当前登陆用户被授予的系统权限

回收系统权限：   REVOKE {privilege | role} FROM {user_name | role_name | PUBLIC}
revoke 权限 on 对象名 from username;
查询权限分配情况
数据字典视图                      描述
ROLE_SYS_PRIVS            角色拥有的系统权限
ROLE_TAB_PRIVS            角色拥有的对象权限
USER_TAB_PRIVS_MADE      查询授出去的对象权限(通常是属主自己查）
USER_TAB_PRIVS_RECD      用户拥有的对象权限
USER_COL_PRIVS_MADE      用户分配出去的列的对象权限
USER_COL_PRIVS_RECD      用户拥有的关于列的对象权限
USER_SYS_PRIVS            用户拥有的系统权限
USER_TAB_PRIVS            用户拥有的对象权限
USER_ROLE_PRIVS       用户拥有的角色
查询已授予的对象权限(即某个用户对哪些表对哪些用户开放了对象权限)
    SQL> SELECT * FROM user_tab_privs_made; 
 查看特定对象下用户所拥有的对象
使用dba_objects视图
与权限安全相关的数据字典表有:
ALL_TAB_PRIVS
ALL_TAB_PRIVS_MADE
ALL_TAB_PRIVS_RECD
DBA_SYS_PRIVS
DBA_ROLES
DBA_ROLE_PRIVS
ROLE_ROLE_PRIVS
ROLE_SYS_PRIVS
ROLE_TAB_PRIVS
SESSION_PRIVS
SESSION_ROLES
USER_SYS_PRIVS
USER_TAB_PRIV
### 三种角色权限详解
#### CONNECT角色
授予最终用户的典型权利，最基本的权力，能够连接到Oracle数据库中，并在对其他用户的表有访问权限时，做SELECT、UPDATE、INSERTT等操作。
 ALTER SESSION --修改会话
CREATE CLUSTER --建立聚簇
CREATE DATABASE LINK --建立数据库链接
CREATE SEQUENCE --建立序列
CREATE SESSION --建立会话
CREATE SYNONYM --建立同义词 
CREATE VIEW --建立视图
#### RESOURCE角色
 授予开发人员的，能在自己的方案中创建表、序列、视图等。
CREATE CLUSTER --建立聚簇
CREATE PROCEDURE --建立过程
CREATE SEQUENCE --建立序列
CREATE TABLE --建表
CREATE TRIGGER --建立触发器
CREATE TYPE --建立类型
#### DBA角色
授予系统管理员的，拥有该角色的用户就能成为系统管理员了，它拥有所有的系统权限。
授权命令 语法： GRANT CONNECT, RESOURCE TO 用户名;
撤销权限 语法： REVOKE CONNECT, RESOURCE FROM 用户名;

select owner, table_name from all_tables a where a.table_name like '%USER%';