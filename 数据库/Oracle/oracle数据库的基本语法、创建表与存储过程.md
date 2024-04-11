# oracle数据库
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
select tablespace_name, table_name from user_tables
where tablespace_name = 'ZXINSYS';               查看表空间下的表
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
### if else
标准sql规范
复制代码

 ```
 if a=..  then
  语句
 end if;
 ```
2、decode函数

DECODE的语法：
```
DECODE(value,if1,then1,if2,then2,if3,then3,...,else)
```
表示如果value等于if1时，DECODE函数的结果返回then1,...,如果不等于任何一个if值，则返回else。
3、case when

```
case when a='1'then 'xxxx'
     when a='2' then 'ssss'
else
　　'zzzzz'
end as
```
注意点： 

1、以CASE开头，以END结尾 
2、分支中WHEN 后跟条件，THEN为显示结果 
3、ELSE 为除此之外的默认情况，类似于高级语言程序中switch case的default，可以不加 
4、END 后跟别名
## 存储过程的创建
Oracle存储过程包含三部分：过程声明，执行过程部分，存储过程异常。

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
## 基本结构

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
## SELECT INTO STATEMENT
将select查询的结果存入到变量中，可以同时将多个列存储多个变量中，必须有一条记录，否则抛出异常(如果没有记录抛出NO_DATA_FOUND)
例子：
```
BEGIN
SELECT col1,col2 into 变量1,变量2 FROM typestruct where xxx;
EXCEPTION
WHEN NO_DATA_FOUND THEN
    xxxx;
END;
```
## 删除存储过程
```
DROP PROCEDURE procedure_name;
```
## 调用存储过程
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



