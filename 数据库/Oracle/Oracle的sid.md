oracle的sid
什么是oracle? 平常所说的 Oracle 或 Oracle 数据库指的是 Oracle 数据库管理系统. Oracle 数据库管理系统是管理数据库访问的计算机软件(Oracle database manager system). 它由 Oracle 数据库和 Oracle 实例(instance)构成(区别mysql,mysql没有实例的概念)
Oracle 实例: 位于物理内存里的数据结构，它由操作系统的多个后台进程和一个共享的内存池所组成,共享的内存池可以被所有进程访问.Oracle 用它们来管理数据库访问.用户如果要存取数据库(也就是硬盘上的文件) 里的数据, 必须通过Oracle实例才能实现, 不能直接读取硬盘上的文件.实际上, Oracle 实例就是平常所说的数据库服务(service) .在任何时刻一个实例只能与一个数据库关联，访问一个数据库；而同一个数据库可由多个实例访问（RAC）

1、一个Oracle数据库系统中可以同时安装几个数据库，每一个数据库对应一个唯一的实例，但是OPS系统除外，可以多个实例同时对一个数据库操作，称为并行服务器
2、只是一个名字，SID即是INSTANCE_NAME，SERVICE_NAMES主要用在监听器中，service_names是对外的服务名，是服务器端使用的，一个库可以设置多个对外服务名。比如你身份证叫王大锤，这个就是SID,但是对外不同圈子你有很多外号，蛋蛋，二狗子，这些对外的称呼就是SERVICE_NAME.
3、NET EASY CONFIG操纵的应该是主机字符串，是为客户端服务的，
一个数据库可以对外公布多个服务名（SERVICE_NAMES）
一个客户端也可以用多个主机字符串连接到同一个数据库服务器上
4、一个OS上可以装多个Oracle数据库（小的概念），每个库可以对外公布多个服务名，都通过init.ora和listener.ora来实现

注意：在mysql中数据库与数据库实例是一对一之间的关系，区别于oracle一对多的概念

Oracle的服务名(ServiceName)查询
SQL> show parameter service_name;

Oracle的SID查询命令：
SQL> select instance_name from v$instance;

查看Oracle版本
SQL> select version from v$instance