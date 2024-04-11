@[toc]
# oracle数据库
## Oracle 数据库、实例、用户、表空间、表之间的关系
![在这里插入图片描述](https://img-blog.csdnimg.cn/19de1721313340a1ae08a0c0a6644a38.png)
### 数据库
数据库是数据集合。Oracle是一种数据库管理系统，是一种关系型的数据库管理系统。
通常情况了我们称的“数据库”，并不仅指物理的数据集合，他包含物理数据、数据库管理系统。也即物理数据、内存、操作系统进程的组合体。
查询当前数据库名：
```
select name from v$database;
```
### 数据库实例

用Oracle官方描述：实例是访问Oracle数据库所需的一部分计算机内存和辅助处理后台进程，是由进程和这些进程所使用的内存(SGA)所构成一个集合。
其实就是用来访问和使用数据库的一块进程，它只存在于内存中。就像Java中new出来的实例对象一样。
我们访问Oracle都是访问一个实例，但这个实例如果关联了数据库文件，就是可以访问的，如果没有，就会得到实例不可用的错误。
实例名指的是用于响应某个数据库操作的数据库管理系统的名称。她同时也叫SID。实例名是由参数instance_name决定的。
查询当前数据库实例名：
```
select instance_name from v$instance;
```
数据库实例名(instance_name)用于对外部连接。在操作系统中要取得与数据库的联系，必须使用数据库实例名。比如我们作开发，要连接数据库，就得连接数据库实例名：
jdbc:oracle:thin:@localhost:1521:orcl(orcl就为数据库实例名)
一个数据库可以有多个实例，在作数据库服务集群的时候可以用到。
### 表空间

Oracle数据库是通过表空间来存储物理表的，一个数据库实例可以有N个表空间，一个表空间下可以有N张表。
有了数据库，就可以创建表空间。
表空间(tablespace)是数据库的逻辑划分，每个数据库至少有一个表空间(称作SYSTEM表空间)。为了便于管理和提高运行效率，可以使用一些附加表空间来划分用户和应用程序。例如：USER表空间供一般用户使用，RBS表空间供回滚段使用。一个表空间只能属于一个数据库。
创建表空间语法：
```
Create TableSpace 表空间名称
DataFile 表空间数据文件路径
Size 表空间初始大小
Autoextendon
```
如：
```
create tablespace db_test
datafile 'D:\oracle\product\10.2.0\userdata\db_test.dbf'
size 50m
autoextend on;
```

查看已经创建好的表空间：

```
select d.username from dba_users d;
```

查看当前用户的缺省表空间：

```
select  username,default_tablespace from user_users;
```
###  表空间与表
查询表空间所有表：select table_name from all_tables where TABLESPACE_NAME='表空间' 表空间名字一定要大写。
查询表所在的表空间：select * from user_tables where table_name=‘表名';表名一定要大写；
### 用户
Oracle数据库建好后，要想在数据库里建表，必须先为数据库建立用户，并为用户指定表空间。
上面我们建好了数据库和表空间，接下来建用户：
创建新用户：
```
CREATE USER 用户名
IDENTIFIEDBY 密码
DEFAULT TABLESPACE 表空间(默认USERS)
TEMPORARY TABLESPACE 临时表空间(默认TEMP)
```

如：
```
CREATE USER wpp IDENTIFIED BY 1234 DEFAULT TABLESPACE SYSTEM TEMPORARY TABLESPACE TEMP;
```
(这里临时表空间不能使用我们创建的db_test，不知为何?)
有了用户，要想使用用户账号管理自己的表空间，还得给它分权限：
```
GRANT CONNECT TO wpp;
GRANT RESOURCE TO wpp;
GRANT dba TO wpp;--dba为最高级权限，可以创建数据库，表等。
```
查看数据库用户：

```
select username from dba_users;
```
为其设置了connect权限之后，就可以登录了
connect wpp
## 基本语法
create database {ORACLE_NAME}　　   --> # 数据库名，一般与ORACLE_SID相，创建数据库
show user 查看当前用户　　
create user username identified by password; 创建新用户
 create tablespace tablespacename datafile 'd:\data.dbf' size xxxm;创建表空间 ，例如
 ```
 create tablespace animal  datafile 'animal.dbf' size 10M;
 ```
tablespacename：表空间的名字
d:\data.dbf'：表空间的存储位置
xxx表空间的大小，m单位为兆(M)
 alter user username default tablespace tablespacename; 将空间分配给用户：   将名字为tablespacename的表空间分配给username 
 grant create session,create table,unlimited tablespace to zxin; 给用户授权

conn username/password;   以自己创建的用户登录，登录之后创建表即可。
select tablespace_name from user_tablespaces;  查询当前用户拥有的所的有表空间
select tablespace_name, table_name from user_tables where tablespace_name = 'ZXINSYS';               查看表空间下的表
select * from tab where tabtype='TABLE'　 查看当前用户下的表
desc dept　　　　　　　　　　　　 察看表dept的结构
quit/exit　　　　　　　　　　　　 退出
clear screen　　　　　　　　　 清除屏幕
select * from v$version;查看当前Oracle的版本
conn system/welcome select * from v$tablespace; 查看表空间
grant create any table to zxinsys;  给用户添加存储过程权限
show errors;  查看上一个语句的错误信息

注意：
是Oracle的对象名字最多是30个字符，不能超过30

例子
create database zxin;
create user zxinsys identified   by  Aa#12345;
create tablespace zxinsys  datafile 'zxinsys.dbf' size 10M;
alter user zxinsys default tablespace zxinsys;
grant create session,create table,unlimited tablespace to zxinsys;
conn zxinsys/Aa#12345;


create tablespace testOracle  datafile 'oracle.dbf' size 1M;
create table testTable(id varchar(3)) tablespace testOracle;
select tablespace_name, table_name from user_tables
where tablespace_name = 'TESTORACLE';
## 基础入门
这个比较简单，常用的增删查改暂不多写
### 创建表
```
CREATE TABLE schema_name.table_name (
    column_1 data_type column_constraint,
    column_2 data_type column_constraint,
    ...
    table_constraint
 );
```
### 插入表
```
INSERT INTO table_name (column_1, column_2, column_3, ... column_n)
VALUES( value_1, value_2, value_3, ..., value_n);
```
### 对列进行操作
增加一列：

   alter table emp4 add test varchar2(10);

修改一列：

   alter table emp4 modify test varchar2(20);

删除一列：

alter table emp4 drop column test;

## 存储过程的创建
Oracle存储过程包含三部分：过程声明，执行过程部分，存储过程异常。

### 基本结构

```
CREATE OR REPLACE PROCEDURE 存储过程名字
(
    参数1 IN NUMBER,
    参数2 IN NUMBER
) IS
变量1 INTEGER :=0;
变量2 DATE;
BEGIN
    --执行体
END 存储过程名字;
```
### 存储过程创建语法
```
create [or replace] procedure 存储过程名（param1 in type，param2 out type）
as
变量1 类型（值范围）;
变量2 类型（值范围）;
Begin
    Select count(*) into 变量1 from 表A where列名=param1；
    If (判断条件) then
       Select 列名 into 变量2 from 表A where列名=param1；
       Dbms_output.Put_line(‘打印信息’);
    Elsif (判断条件) then
       Dbms_output.Put_line(‘打印信息’);
    Else
       Raise 异常名（NO_DATA_FOUND）;
    End if;
Exception
    When others then
       Rollback;
End;
```
 param1为输入参数，param2为输出参数，在as里分别设置param1和param2的值范围，如：
```
create or replace procedure zxinsys.proc_insert_operpwdhistory (param1 in varchar2,param2 in varchar2，param3 in varchar2 default 'pwd', param4 out varchar2 default '0')
as
  iparam1   number(10);
  iparam2   number(10);
  iparam3   number(10);
  iparam4   number(10);
```
select count(*) into 变量为将获得的值传入参数，在后面可能会调用
### 无参存储过程语法
```
create or replace procedure NoParPro
 as  //声明
 ;
 begin // 执行
 ;
 exception//存储过程异常
 ;
 end;
```
### 带参存储过程实例
```
create or replace procedure queryempname(sfindno emp.empno%type)
as
   sName emp.ename%type;
   sjob emp.job%type;
begin
       ....
exception
       ....
end;
```
### 带参数存储过程含赋值方式
```
create or replace procedure runbyparmeters
    (isal in emp.sal%type,
     sname out varchar,
     sjob in out varchar)
 as
    icount number;
 begin 
      select count(*) into icount from emp where sal>isal and job=sjob;
      if icount=1 then 
        ....
      else
       ....
     end if;
exception
     when too_many_rows the
     DBMS_OUTPUT.PUT_LINE('返回值多于1行');
     when others then 
     DBMS_OUTPUT.PUT_LINE('在RUNBYPARMETERS过程中出错！');
end;
```
其中参数IN表示输入参数，是参数的默认模式。
OUT表示返回值参数，类型可以使用任意Oracle中的合法类型。
OUT模式定义的参数只能在过程体内部赋值，表示该参数可以将某个值传递回调用他的过程
IN OUT表示该参数可以向该过程中传递值，也可以将某个值传出去。

###  事务嵌套
一般使用savepoint来进行事务部分回滚
SAVEPOINT t1;
回滚rollback to savepoint t1
### 查看存储过程
SELECT text
    FROM user_source
   WHERE NAME = '存储过程名'
ORDER BY line;
###  删除存储过程
```
DROP PROCEDURE procedure_name;
```
### 调用存储过程
exec pro_name(参数1..);
call pro_name(参数1..);
 exec是sqlplus命令，只能在sqlplus中使用；call为SQL命令，没有限制.
存储过程没有参数时,exec可以直接跟过程名（可以省略()），但call则必须带上().
另一种方法
```
begin
  procedure;--procedure是存储过程名
end;
```

Oracle程序调用过程让我们来看看如何调用上面创建的过程。参考以下示例代码 
```
BEGIN
   insertuser(101,'Maxsu');
   dbms_output.put_line('record inserted successfully');
END;
```
返回参数
```
begin
  queryEmpInformation(eno => 7839,
                      pename => :pename,
                      psal   => :psal,
                      pjob   => :pjob);   --  =>也表示赋值的意思
end;
call
exec
```
一般使用的方法：pjob可以作为输入参数直接放入，也可以直接打印结果
```
set serveroutput on;
declare
pjob number;
begin
DBMS_OUTPUT.PUT_LINE('测试空格情况');
zxinsys.proc_remove_res_definition('');
DBMS_OUTPUT.PUT_LINE('测试null情况');
zxinsys.proc_remove_res_definition(null);
end;
/
show errors procedure zxinsys.proc_remove_res_definition;
```
## 常用语句
to_char(sysdate,'YYYY.MM.DD HH24:MI:SS')
insert into 表名 select * from 表 where 条件；
v_date date 日期字段
v_date := to_date(v_updatedate,'YYYY-MM-DD hh24:mi:ss');将字符串转化为时间格式
as与is的区别
循环语句
for  vRows in  
LOOP
循环体
end LOOP
cursor

### Case When
格式语法
```
case
    when 条件1 then action1
    when 条件2 then action2
    when 条件3 then action3
    when 条件N then actionN
    else action
end
```
判断现在是几月

```
SQL> select case substr('20181118',5,2)
  2  when '08' then '8yue'
  3  when '09' then '9yue'
  4  when '10' then '10yue'
  5  when '11' then '11yue'
  6  when '12' then '12yue'
  7  else 'other'
  8  end
  9  from dual;

CASESUBSTR('201
---------------
11yue

```
选出等级的值

```
select grade from
(select sno,case
 when score >=90 then 'A'
 when score >=80 then 'B'
 when score >=70 then 'C'
 when score >=60 then 'D'
 else 'F'
 end as grade
  9   from score);

GRADE
----------
D
C
B
A
```
GROUP BY CASE WHEN 用法
```
SELECT  
CASE WHEN salary <= 500 THEN '1'  
WHEN salary > 500 AND salary <= 600  THEN '2'  
WHEN salary > 600 AND salary <= 800  THEN '3'  
WHEN salary > 800 AND salary <= 1000 THEN '4'  
ELSE NULL END salary_class, -- 别名命名
COUNT(*)  
FROM    Table_A  
GROUP BY  
CASE WHEN salary <= 500 THEN '1'  
WHEN salary > 500 AND salary <= 600  THEN '2'  
WHEN salary > 600 AND salary <= 800  THEN '3'  
WHEN salary > 800 AND salary <= 1000 THEN '4'  
ELSE NULL END;  
```
### 拼接字符串
1、拼接字符串

1）可以使用“||”来拼接字符串

1 select '拼接'||'字符串' as str from dual 
2）通过concat()函数实现

1 select concat('拼接', '字符串') as str from dual 
注：oracle的concat函数只支持两个参数的方法，即只能拼接两个参数，如要拼接多个参数则嵌套使用concat可实现，如：

1 select concat(concat('拼接', '多个'), '字符串') from dual 
concat与EXECUTE IMMEDIATE 用法
需要注意：
在Oracle中，CONCAT函数将只允许您将两个值连接在一起。如果需要连接多个值，那么我们可以嵌套多个CONCAT函数调用。
```
SELECT CONCAT(CONCAT('A', 'B'),'C')
FROM dual;
-- Result: 'ABC'
```
以上这个例子将3个值连接在一起并返回'ABC'字符串值。

```
SELECT CONCAT(CONCAT(CONCAT('A', 'B'),'C'),'D')
FROM dual;
- Result: 'ABCD'
```
这个例子将连接4个值并返回'ABCD'。

连接单引号
由于CONCAT函数中的参数由单引号分隔，因此如何在CONCAT函数的结果中添加单引号字符并不简单。

我们来看一个相当简单的例子，它展示了如何使用CONCAT函数向结果字符串添加单引号。

```
SELECT CONCAT('Let''s', ' learn Oracle')
FROM dual;
-- Result: Let's learn Oracle
```
由于参数包含在单引号中，因此在引号内使用2个额外的单引号来表示生成的连接字符串中的单引号。

### EXECUTE IMMEDIATE
它解析并马上执行动态的SQL语句或非运行时创建的PL/SQL块.动态创建和执行SQL语句性能超前，EXECUTE IMMEDIATE的目标在于减小企业费用并获得较高的性能，较之以前它相当容易编码
execute immediate
例如：
```
 sqlstr:='select * from zxinsys.oper_information2 where operid =' || '1';
        execute immediate sqlstr;
```
### for loop的用法
为了对游标进行遍历，可以使用 FOR LOOP 语句实现，语法如下：
FOR record IN cursor_name
LOOP
  process_record_statements;
END LOOP;
对游标进行 FOR LOOP 遍历时，省去了 %ROWTYPE 的声明，循环每次从查询结果中取出一行结果，当取完后，跳出循环。

### 打印输出
dbms_output.put_line('end')

### oracle查询某表是否存在,某字段是否存在某表
select count(0) as v_max_modify
from user_tab_columns   
where UPPER(column_name)='max_modify' AND TABLE_NAME = 'zxin_pwdsafeset';
SELECT table_name, column_name, data_type FROM all_tab_cols WHERE table_name = 'ZXIN_PWDSAFESET' AND UPPER(column_name) ='MAX_MODIFY';
SELECT * FROM user_tab_columns WHERE TABLE_NAME = 'zxin_pwdsafeset';
select * from dba_tab_columns where Table_Name='zxin_pwdsafeset';
select * from all_tab_columns ;
SELECT count(*) FROM zxin.zxin_pwdsafeset;
select count(1) as b from all_tables where TABLE_NAME = 'zxin_pwdsafeset' and OWNER='select user from zxin';

注意一定要大写，最后能成功的：
SELECT table_name, column_name, data_type FROM all_tab_cols WHERE table_name = 'ZXIN_PWDSAFESET' AND UPPER(column_name) ='MAX_MODIFY';
### 拼接语句concat
CONCAT()函数在Oracle中可以用于将两个字符串连接在一起，那么CONCAT()函数的语法及使用方法是什么呢？下面一起来看看。
CONCAT()函数语法
CONCAT( string1, string2 )
string1：第一个要连接的字符串。
string2：第二个要连接的字符串。
返回值
CONCAT函数返回string1连接string2后的一个字符串值。
适用版本
CONCAT()函数可用于以下版本的Oracle：

Oracle 12c、 Oracle 11g、 Oracle 10g、 Oracle 9i

在Oracle中，CONCAT函数将只允许您将两个值连接在一起。如果需要连接多个值，那么我们可以嵌套多个CONCAT函数调

### oracle中sql%rowcount的作用
sql%rowcount来判断是否更新了记录的状态
sql%rowcount只会记录未被提交的最后一条SQL语句的影响行数。这点很重要，如果想统计多个sql的合计影响行数，就必须在每个sql后面，用一个变量保存当前的sql%rowcount。
### commit
commit（提交）的作用
在数据库的插入、删除和修改操作时，只有当事务在提交到数据库时才算完成。在Oracle 数据库中，
在事务提交前，只有操作数据库的这个人才能有权看到所做的事情，别人只有在最后提交完成时才能看见。
如果没有提交的话，则数据会保存到内存中去，但是并没有保存到数据文件。
它执行的时候，你不会有什么感觉。commit在数据库编程的时候很常用，当你执行DML操作时，数据库并不会立刻修改表中数据，这时你需要commit，数据库中的数据就立刻修改了，如果在没有commit之前，就算你把整个表中数据都删了，如果rollback的话，数据依然能够还原。
测试语句
```
 begin
<<fst_loop>>
loop

   dbms_output.put_line('v_rlt = '||v_rlt);
   insert into wpp.user1 values(v_rlt);
   dbms_output.put_line('influence row number'||sql%rowcount);
   select count(*) into vcount  from wpp.user1;
   dbms_output.put_line('select='||vcount);
   v_rlt:=v_rlt+1;
   exit fst_loop when v_rlt > 10;
  rollback 或者 commit;
end loop;
dbms_output.put_line('nums:'||sql%rowcount);
 dbms_output.put_line('LOOP循�m~N�已�[m~O�S�~_�A');
end;
/
select * from wpp.user1;
```
### 日期与函数
SELECT EXTRACT( YEAR FROM SYSDATE ) FROM DUAL;
select   extract (month from sysdate)*30.1+extract (day from sysdate) day from  dual;
select   floor(extract (hour from sysdate)*30.1 +extract (day from sysdate))day from  dual;
## Oracle 动态游标实现动态sql循环
### CURSOR <游标名称> IS <游标名称>%ROWTYPE;
```
复制内容到剪贴板/ 程序代码
CURSOR <游标名称> IS
<游标名称>%ROWTYPE;
BEGIN
  OPEN <游标名称>
  LOOP
    FETCH <游标名称> INTO ;
    EXIT WHEN <游标名称>%NOTFOUND;

    <其它要执行的代码>
  END LOOP;
  CLOSE <游标名称>;
END <存储过程名称>;
/
```
实例：
```
create or replace procedure P_TEST_SQL is
TYPE ref_cursor_type IS REF CURSOR;  --定义一个动态游 
tablename varchar2(200) default 'ess_client';
v_sql varchar2(1000);
mobile varchar2(15);
usrs ref_cursor_type;
begin
  --使用连接符拼接成一条完整SQL
  v_sql := 'select usrmsisdn from '||tablename||' where rownum < 11';  
  --打开游标
  open usrs for v_sql ;
  loop
      fetch usrs into mobile;
      exit when usrs%notfound;
      insert into tmp(usrmsisdn) values(mobile);
  end loop;
  close usrs;
  commit;
end P_TEST_SQL;
```
### Cursor FOR Loop格式
```
Create or REPLACE PROCEDURE <存储过程名称> IS
CURSOR <游标名称> IS
BEGIN
  FOR IN <游标名称>
  LOOP
    <其它要执行的代码>
  END LOOP;
END <存储过程名称>;
/
```
实例
```
TRUNCATE TABLE loop_test;
DECLARE
CURSOR ao_cur IS
   Select SUBSTR(object_name,1,5) FIRSTFIVE
   FROM all_objs
   Where SUBSTR(object_name,1,5) BETWEEN 'N' AND 'W';
BEGIN
  FOR ao_rec IN ao_cur LOOP
    Insert INTO loop_test VALUES (ao_rec.firstfive);
  END LOOP;
  COMMIT;
END;
/

Select COUNT(*) FROM loop_test;
```
## oracle游标的用法
Oracle游标是通过关键字CURSOR的来定义一组Oracle查询出来的数据集，类似数组一样，把查询的数据集存储在内存当中，然后通过游标指向其中一条记录，通过循环游标达到循环数据集的目的。
Oracle游标可以分为显式游标和隐式游标两种之分。

显式游标：指的是游标使用之前必须得先声明定义，一般是对查询语句的结果事进行定义游标，然后通过打开游标循环获取结果集内的记录，或者可以根据业务需求跳出循环结束游标的获取。循环完成后，可以通过关闭游标，结果集就不能再获取了。全部操作完全由开发者自己编写完成，自己控制。

隐式游标：指的是PL/SQL自己管理的游标，开发者不能自己控制操作，只能获得它的属性信息。
### 声明游标：
声明游标是给游标命名并给游标关联一个查询结果集，具体声明语法如下：
```
declare cursor cursor_name(游标名)
is select＿statement(查询语句);
```
### 打开游标：
游标声明完，可以通过打开游标打开命令，初始化游标指针，游标一旦打开后，游标对应的结果集就是静态不会再变了，不管查询的表的基础数据发生了变化。打开游标的命令如下：
```
open cursor_name;
```
### 读取游标中数据：

读取游标中的数据是通过fetch into语句完成，把当前游标指针指向的数据行读取到对应的变量中（record 变量）。游标读取一般和循环LOOP一起使用，用于循环获取数据集中的记录。

```
fetch cursor_name into record变量
```
### 关闭游标：

游标使用完，一定要关闭游标释放资源。关闭后，该游标关联的结果集就释放了，不能够再操作了，命令如下：

```
close cursor_name;
```
### 案例
创建一个游标循环打印学生信息表中学生基本信息，代码如下：
```
declare
  --定义游标
  cursor cur_xsjbxx is
    select * from stuinfo order by stuid;
  --定义记录变量  
  ls_curinfo cur_xsjbxx%rowtype;
begin
  open cur_xsjbxx;--打开游标
  loop
    FETCH cur_xsjbxx
      INTO ls_curinfo;--获取记录值
    EXIT WHEN cur_xsjbxx%NOTFOUND;
  
    dbms_output.put_line('学号：' || ls_curinfo.stuid || ',姓名：' ||
                         ls_curinfo.STUNAME);
  end loop;
  close cur_xsjbxx;--关闭游标
end;
```
## 调试技巧
show errors procedure 存储过程名字;    可以列出存储过程出现的问题
SET SETVEROUTPUT ON 加上这句才能使得输出语句生效

## 注意
您不能在 PL/SQL 块中将 DDL 作为静态 SQL 发出。如果要将这些命令放在 PL/SQL 块中，则需要使用动态 SQl

## Oracle创建视图(View)
视图：是基于一个表或多个表或视图的逻辑表，本身不包含数据，通过它可以对表里面的数据进行查询和修改。视图基于的表称为基表，Oracle的数据库对象分为五种：表，视图，序列，索引和同义词。
视图是存储在数据字典里的一条select语句。通过创建视图可以提取数据的逻辑上的集合或组合。
视图的优点：
1.对数据库的访问，因为视图可以有选择性的选取数据库里的一部分。
2.用户通过简单的查询可以从复杂查询中得到结果。
3.维护数据的独立性，试图可从多个表检索数据。
4.对于相同的数据可产生不同的视图。
视图分为简单视图和复杂视图：
1、简单视图只从单表里获取数据，复杂视图从多表；
2、简单视图不包含函数和数据组，复杂视图包含；
3、简单视图可以实现DML操作，复杂视图不可以。
语法结构：创建视图

```
CREATE [OR REPLACE] [FORCE|NOFORCE] VIEW view_name
    [(alias[, alias]...)]

AS subquery
     [WITH CHECK OPTION [CONSTRAINT constraint]]
     [WITH READ ONLY]
```
### 语法解析：

OR REPLACE    ：若所创建的试图已经存在，则替换旧视图；

FORCE：不管基表是否存在ORACLE都会自动创建该视图(即使基表不存在，也可以创建该视图，但是该视图不能正常使用，当基表创建成功后，视图才能正常使用)；

NOFORCE  ：如果基表不存在，无法创建视图，该项是默认选项(只有基表都存在ORACLE才会创建该视图)。

alias：为视图产生的列定义的别名；

subquery  ：一条完整的SELECT语句，可以在该语句中定义别名；

WITH CHECK OPTION  ：插入或修改的数据行必须满足视图定义的约束；

WITH READ ONLY       ：默认可以通过视图对基表执行增删改操作，但是有很多在基表上的限制(比如：基表中某列不能为空，但是该列没有出现在视图中，则不能通过视图执行insert操作)，WITH READ ONLY说明视图是只读视图，不能通过该视图进行增删改操作。现实开发中，基本上不通过视图对表中的数据进行增删改操作。
### 使用drop view 命令删除视图
格式：
drop view 视图名
### 利用视图操作基本表
---添加数据行（直接影响基本表）
insert into tb_user_view1 values(8,'insert',5)
---修改数据行（影响基本表）
update tb_user_view1 set username='update' where bh=3
---删除数据行（直接影响基本表）
delete from tb_user_view1 where bh=2
多表关联创建视图
格式：CREATE OR REPLACE FORCE VIEW 视图名（列表名1，列表名2，……，列表名n）as select  查询子句
例子：
CREATE OR REPLACE FORCE VIEW  tb_username_address (bh, username, address_id, address) AS
  SELECT u.bh,u.username,a.bh,a.address FROM tb_user u,tb_address a WHERE  u.address_id=a.bh
修改视图
格式：CREATE OR REPLACE FORCE VIEW 视图名（列表名1，列表名2，……，列表名n）as select  查询子句
例子：
CREATE OR REPLACE FORCE VIEW  tb_username_address (bh, username, address_id, address) AS
  SELECT u.bh,u.username,a.bh,a.address FROM tb_user u,tb_address a WHERE  u.address_id=a.bh and a.address='xinxiang'
drop view 命令删除视图
## 出现的问题
ORA-00937: not a single-group group function
SCOTT@PROD> select deptno,sum(sal) from emp;
select deptno,sum(sal) from emp
ERROR at line 1:
ORA-00937: not a single-group group function
原因：这句话不会运行，因为deptno要求每行都显示，而sum要求统计后再显示，违反了原则。在有组函数的select中，不是组函数的列，一定要放在group by子句中。
正确语句：select deptno,sum(sal) from emp group by deptno;





