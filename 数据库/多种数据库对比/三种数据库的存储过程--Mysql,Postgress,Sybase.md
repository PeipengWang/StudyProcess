
# 三种数据库的存储过程
存储过程的创建和调用
存储过程就是具有名字的一段代码，用来完成一个特定的功能。
创建的存储过程保存在数据库的数据字典中。
Mysql存储例程实际包含了存储过程和存储函数，它们被统称为存储例程。
其中存储过程主要完成在获取记录或插入记录或更新记录或删除记录，即完成select insert delete update等的工作。而存储函数只完成查询的工作，可接受输入参数并返回一个结果。

## mysql
### 基本语法
创建mysql存储过程、存储函数
create procedure 存储过程名(参数)
存储过程简单例子
```
DELIMITER // 
CREATE PROCEDURE proc1(OUT s int)
BEGIN
  SELECT COUNT(*) INTO s FROM user;
END //
DELIMITER ;

```
1）这里需要注意的是DELIMITER//和DELIMITER;两句， DELIMITER是分割符的意思，因为MySQL默认以";"为分隔 符，如果我们没有声明分割符，那么编译器会把存储过程当成SQL语句进行处理，则存储过程的编译过程会报错，所以要事先用DELIMITER关键字申明当 前段分隔符，这样MySQL才会将";"当做存储过程中的代码，不会执行这些代码，用完了之后要把分隔符还原。
2）存储过程根据需要可能会有输入、输出、输入输出参数，分别对应为IN、OUT、INOUT，这里有一个输出参数s，类型是int型，如果有多个参数用","分割开。
3）过程体的开始与结束使用BEGIN与END进行标识。
### 存储过程的输入参数
MySQL存储过程的参数用在存储过程的定义，共有三种参数类型,IN,OUT,INOUT,形式如：
CREATEPROCEDURE 存储过程名([[IN |OUT |INOUT ] 参数名 数据类形...])
IN 输入参数:表示该参数的值必须在调用存储过程时指定，在存储过程中修改该参数的值不能被返回，为默认值
OUT 输出参数:该值可在存储过程内部被改变，并可返回
INOUT 输入输出参数:调用时指定，并且可被改变和返回

下面来学习下存储过程中的参数，先看下存储过程中的参数形式，如下：
 存储过程中是空的参数列表
```
CREATE PROCEDURE proc1 ()
```
储过程中有一个输出参数，名称为varname，后面是跟数据类型DATA-TYPE,IN参数是默认的，因此可以省略不写
```
 CREATE PROCEDURE proc1 (IN varname DATA-TYPE)
 DELIMITER //
CREATE PROCEDURE `proc_IN` (IN var1 INT)
BEGIN
    SELECT var1 + 2 AS result;
END//
```
输出参数
```
DELIMITER //
CREATE PROCEDURE `proc_OUT` (OUT var1 VARCHAR(100))
BEGIN
　　SET var1 = 'This is a test';
END //
CREATE PROCEDURE proc1 (OUT varname DATA-TYPE)
varname既是输入参数也是输出参数
```
输入又输出
```
CREATE PROCEDURE proc1 (INOUT varname DATA-TYPE)
DELIMITER //
CREATE PROCEDURE `proc_INOUT` (OUT var1 INT)
BEGIN
　　SET var1 = var1 * 2;
END //
```
### 存储过程中定义的局部变量
Ⅰ. 变量定义
局部变量声明一定要放在存储过程体的开始
DECLAREvariable_name [,variable_name...] datatype [DEFAULT value];
其中，datatype为MySQL的数据类型，如:int, float, date,varchar(length)
例如:
```
DECLARE l_int int unsigned default 4000000;
```
Ⅱ. 变量赋值
SET 变量名 = 表达式值 [,variable_name = expression ...]
```
SET @y='Goodbye Cruel World';
SELECT @y;
+---------------------+
|     @y              |
+---------------------+
| Goodbye Cruel World |
+---------------------+
```
Ⅲ. 用户变量
### MySQL存储过程的调用
用call和你过程名以及一个括号，括号里面根据需要，加入参数，参数包括输入参数、输出参数、输入输出参数。
```
CALL stored_procedure_name (param1, param2, ....)
CALL procedure1(10 , 'string parameter' , @parameter_var);
```
其中不加@的参数为输入参数，加上@param的参数为输出参数
可以通过select  @param的方式获得输出参数值
### MySQL存储过程的查询
我们像知道一个数据库下面有那些表，我们一般采用show tables进行查看。那么我们要查看某个数据库下面的存储过程，是否也可以采用呢？答案是，我们可以查看某个数据库下面的存储过程，但是是令一钟方式。
我们可以用
```
select name from mysql.proc where db=’数据库名’;
```
或者
```
select routine_name from information_schema.routines where routine_schema='数据库名';
```
或者
```
showprocedure status where db='数据库名';
```
进行查询。
如果我们想知道，某个存储过程的详细，那我们又该怎么做呢？是不是也可以像操作表一样用describe 表名进行查看呢？
答案是：我们可以查看存储过程的详细，但是需要用另一种方法：
SHOWCREATE PROCEDURE 数据库.存储过程名;
就可以查看当前存储过程的详细。
### MySQL存储过程的修改
可以用ALTER的语法去修改存储过程的主要特征和参数，要修改其存储过程的主体部分的话，必须要先删除然后再重建。比如下面修改存储过程num_from_employee的定义。将读写权限改为MODIFIES SQL DATA，并指明调用者可以执行。代码执行如下：
```
ALTER PROCEDURE num_from_employee
MODIFIES SQL DATA SQL SECURITY INVOKER ;
```
更改用CREATE PROCEDURE 建立的预先指定的存储过程，其不会影响相关存储过程或存储功能。
### MySQL存储过程的删除
删除一个存储过程比较简单，和删除表一样：
```
DROP PROCEDURE procedure_name
```
从MySQL的表格中删除一个或多个存储过程。

## Postgresql
### 基本语法
```
CREATE [OR REPLACE] FUNCTION function_name (arguments)
RETURNS return_datatype AS $variable_name$
  DECLARE
    declaration; 
    [...]
  BEGIN
    < function_body >
    [...]
    RETURN { variable_name | value }
  END; LANGUAGE plpgsql;
```
function_name：指定函数的名称。
[OR REPLACE]：是可选的，它允许您修改/替换现有函数。
RETURN：它指定要从函数返回的数据类型。它可以是基础，复合或域类型，或者也可以引用表列的类型。
function_body：function_body包含可执行部分。plpgsql：它指定实现该函数的语言的名称。
### 调用方法
```
SELECT function_name(1);
SELECT * FROM function_name(1);
```

## sybase 存储过程修改数据库 
### 创建存储过程
create proc procedure_name as begin SQL_statements [return] end
### 执行存储过程
语法:  [exec[ute]] procedure_name [参数]
### 查看
查看用户自建的存储过程：
select name from sysobjects where type="P"
go
查看创建存储过程的源代码，使用： sp_helptext procedure_name
sp_helptext sp_show_stu
go
### 重新命名存储过程 
语法：sp_rename old_name , new_name 
例：将已创建的存储过程reports_1改名为reports_1b： exec sp_rename reports_1, report_lb
###  删除存储过程 
语法：drop proc procedure_name 
例： 删除已创建的存储过程reports： drop proc reports
### 存储过程中的参数
输入参数(Input Parameters) 是指由调用程序向存储过程 提供的变量值。它们在创建存储过程语句中被定义，而 在执 行该存储过程语句中给出相应的变量值。 使用输入 参数的优 点是使存储过程得更加灵活。
 语法： create proc procedure_name 语法 (@parameter_name datatype [, @parameter_name datatype……]) as begin SQL_statements return end
 例如：
create proc sp_show_stu1
(@sno char(7)) as
begin
    select * from STUDENT where SNO = @sno return
end
go
 在调用程序的执行存储过程命令中，将相应的值传递输入参数：
 exec sp_show_stu1 @sno='9302303'
go
exec proc_author_addr @lname = 'Green' 或 exec proc_author_addr 'Green?? end



