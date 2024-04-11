# 使用profile文件实现用户口令限制与资源限制
Proflie是口令限制,资源限制的命名集合.建立oracle数据库时,oracle会自动建立名为DEFAULT的PROFILE,初始化的DEFAULT没有进行任何口令和资源限制.使用PROFILE有以下一些主要事项.
a,建立PROFILE时,如果只设置了部分口令或资源限制选项,其他选项会自动使用默认值(DEFAULT的相应选项)
b,建立用户时,如果不指定PROFILE选项,oracle会自动将DEFAULT分配给相应的数据库用户.
c,一个用户只能分配一个PROFILE.如果要同时管理用户的口令和资源,那么在建立PROFILE时应该同时指定口令和资源选项.
d,使用PROFILE管理口令时,口令管理选项总是处于被激活状态,但如果使用PROFILE管理资源,必须要激活资源限制.
1.查看当前存在的profile文件
select distinct profile from dba_profiles;
查看指定profile文件中各资源的限制情况：
select resource_name,limit from dba_profiles where profile='DEFAULT';
2.修改现在profile文件中资源选项：
alter profile default limit FAILED_LOGIN_ATTEMPTS 1 PASSWORD_LOCK_TIME 3;
3.创建一个新的profile文件：
CREATE PROFILE lock_accout LIMIT FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 10;
4.让用户使用新的profile文件：
alter user test profile lock_accout;
5.查看用户当前使用的profile文件
select username,profile from dba_users;
6.使用profile文件限制用户对资源的使用;
必须先激活资源限制:
alter system set resource_limit=TRUE scope=memory;
对资源限制做修改:
alter profile lock_accout limit cpu_per_session 5000;
7.删除profile
drop profile lock-accout;
8.删除profile并将使用当前profile的用户profile改为default
drop profile lock_accout cascade;
9，以下列出所有profile相关参数内容以便于参考:
FAILED_LOGIN_ATTEMPTS:用于指定联系登陆的最大失败次数.
PASSWORD_LOCK_TIME:用于指定帐户被锁定的天数.
PASSWORD_LIFE_TIME:用于指定口令有效期
PASSWORD_GRACE_TIME:用于指定口令宽限期.
PASSWORD_REUSE_TIME:用于指定口令可重用时间.
PASSWORD_REUSE_MAX;用于指定在重用口令之前口令需要改变的次数.
PASSWORD_VERIFY_FUNCTION;是否校验口令(校验将值改为VERIFY_FUNCTION)
CPU_PER_SESSION:用于指定每个会话可以占用的最大CPU时间.
LOGICAL_READS_PER_SESSON:用于指定会话的最大逻辑读取次数.
PRIVATE_SGA:用于指定会话在共享池中可以分配的最大总计私有空间.需要注意,该选项只使用与共享服务器模式.
COMPOSITE_LIMIT:用于指定会话的总计资源消耗(单位:服务单元).
CPU_PER_CALL:限制每次调用(解析,执行或提取数据)可占用的最大CPU时间(单位:百分之一秒)
LOGICAL_READS_PER_CALL:用于限制每次调用的最大逻辑I/O次数.
SESSIONS_PER_USER:用于指定每个用户的最大并发会话个数.
CONNECT_TIME:用于指定会话的最大连接时间.
IDLE_TIME:用于指定会话的最大空闲时间.
