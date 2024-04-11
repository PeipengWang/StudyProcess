# 用户与表空间

什么是用户的表空间配额（User tablespace Quota）？

用户的表空间配额也叫表空间限额，指的是用户可以使用指定表空间的最大大小。控制用户的表空间配额也就等于控制用户所占用的表空间大小。在默认情况下，需要对用户赋予RESOURCE角色，虽然该角色没有UNLIMITED TABLESPACE权限，但是对用户赋予了该角色后，系统将会默认给用户赋予UNLIMITED TABLESPACE的系统权限，因此，新建的用户对所有表空间都是没有配额的，即不受空间的限制。表空间配额可以在创建用户的时候指定，也可以在创建用户后再修改用户的配额。关于表空间配额需要注意以下几点：

①　DBA用户是具有UNLIMITED TABLESPACE的权限的，若是回收DBA角色，则会导致UNLIMITED TABLESPACE权限被连带回收，易引起生产事故，所以，在回收DBA角色时需特别注意。
②　当用户使用空间超出限额的时候会报“ORA-01536”和“ORA-01950”的错误。
③　目标用户必须不能含有UNLIMITED TABLESPACE的系统权限，否则空间配额对用户的设置无效，也就会出现在DBA_TS_QUOTAS中的BYTES大于MAX_BYTES的情况。
Oracle 官网对quota的定义如下：
A limit on a resource, such as a limit on the amount of database storage used by a database user. A database administrator can set  quotas for each Oracle Database .
有关Oracle Quota 这块，只在Oracle 的安全管理这块搜到了一些内容。
Managing Security for Oracle Database Users
## 查看用户的默认表空间
```
select username, default_tablespace defspace from dba_users;
```
## 创建用户限额
```
create user 用户名
identified by 密码
default tablespace myspace
quota 10M on myspace
quota 5M on system
temporary tablespace temp
profile default
account unlock;
```
## 查看用户配额信息
相关表
dba_ts_quotas(查看所有用户的表空间配额)
user_ts_quotas(查看当前用户表空间配额)
```
select tablespace_name,username,max_bytes from  DBA_TS_QUOTAS where username='ANQING';
select TABLESPACE_NAME,USERNAME,BYTES/1024,MAX_BYTES/1024/1024 from dba_ts_quotas;//转化单位M，此时无限空间不好看
```
## 修改用户限额
alter user itme quota 50M on users;
alter user 用户名称 quota 256m on 表空间;
## 回收磁盘配额
alter user rose quota 0 on users;

## 注意点：
1、当在创建表空间对象的时候，出现“超出表空间的空间限量”错误提示的处理思路。这种情况一般是由很多种情况所造成的。笔者在遇到这种问题的时候，一般的处理思路是，先查看该用户的磁盘配额，看其是否满足空间上的需求。若该用户没有磁盘配额管理的话，或者磁盘配额大小符合既定需求的话，则建议查看表空间的大小，看是否是因为表空间不足所造成的。若前两步还是不能够解决问题的，则看看表空间的管理方式。若是数据字典管理模式的话，则把他改为本地管理模式，一般就可以解决问题了。 　　
2、若数据库中存放着不同办事处或者不同分公司的数据时，笔者建议对Oracle数据库进行磁盘限额。这可以有效的防止因为硬盘空间的不足导致整个数据库的崩溃。而若对用户进行磁盘配额管理的话，那最多只是某个办事处(某个用户)数据库操作出现故障而已。这也是提高Oracle数据库可用性的一个有效手段。 　　经查,表空间跟表空间限额两个值是不一样的. 　　推测按默认的话oracle应该会给每个用户分配一个默认的表空间限额,具体比例待查,但这比例肯定远小于100%. 　　所以说分配了400M的表空间未必能存储400M的数据. 　　解决办法如下: 　　查看用户表空间的限额 　　select * from user_ts_quotas; 　　max_bytes字段就是了 　　-1是代表没有限制,其它值多少就是多少了.

