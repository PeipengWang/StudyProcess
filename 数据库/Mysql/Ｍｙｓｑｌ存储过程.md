@[TOC](Mysql存储过程大全)
# Mysql存储过程

## 基本使用
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
show procedure status where db='数据库名';
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

## 创建存储过程细节
### 声明语句结束符，可以自定义:
DELIMITER $$
或
DELIMITER //
### 声明存储过程基本定义:
CREATE PROCEDURE demo_in_parameter(IN p_in int)
MySQL存储过程的参数用在存储过程的定义，共有三种参数类型,IN,OUT,INOUT,形式如：
CREATEPROCEDURE 存储过程名([[IN |OUT |INOUT ] 参数名 数据类形...])
IN 输入参数：表示调用者向过程传入值（传入值可以是字面量或变量）
OUT 输出参数：表示过程向调用者传出值(可以返回多个值)（传出值只能是变量）
INOUT 输入输出参数：既表示调用者向过程传入值，又表示过程向调用者传出值（值只能是变量）
在存储过程中：
可以通过select p_inout 来查询
也可以用 set p_inout = n; 来赋值

### 存储过程开始和结束符号:
BEGIN .... END
### 变量
定义:
DECLARE l_int int unsigned default 4000000;
局部变量声明一定要放在存储过程体的开始：
例如:
DECLARE l_int int unsigned default 4000000;
DECLARE l_numeric number(8,2) DEFAULT 9.95;
DECLARE l_date date DEFAULT '1999-12-31'; 
DECLARE l_datetime datetime DEFAULT '1999-12-31 23:59:59'; 
DECLARE l_varchar varchar(255) DEFAULT 'This will not be padded';

变量赋值
SET 变量名 = 表达式值 [,variable_name = expression ...]

用户变量
在MySQL客户端使用用户变量:
mysql > SELECT 'Hello World' into @x;
SET @y='Goodbye Cruel World';
## 存储过程控制语句
###  变量作用域
内部的变量在其作用域范围内享有更高的优先权，当执行到 end。变量时，内部变量消失，此时已经在其作用域外，变量不再可见了，应为在存储过程外再也不能找到这个申明的变量，但是你可以通过 out 参数或者将其值指派给会话变量来保存其值
### 条件语句
if-then-else 语句
例如：
```
 DELIMITER //
CREATE PROCEDURE proc2(IN parameter int)
 begin
   declare var int;
   set var=parameter+1;
   if var=0 then
     insert into t values(17);
   end if;
   if parameter=0 then
     update t set s1=s1+1;
   else
     update t set s1=s1+2;
   end if;
 end;
//
DELIMITER ;
```
case-when语句
```
DELIMITER //
CREATE PROCEDURE proc3 (in parameter int)
  begin
     declare var int;
     set var=parameter+1;
     case var
     when 0 then
       insert into t values(17);
     when 1 then
       insert into t values(18);
     else
       insert into t values(19);
     end case;
  end;
//
mysql > DELIMITER ;
```
### 循环语句
while ···· end while
```
DELIMITER //
CREATE PROCEDURE proc4()
begin
  declare var int;
  set var=0;
  while var<6 do
    insert into t values(var);
    set var=var+1;
  end while;
end;
//
DELIMITER ;
```
2、repeat···· end repeat

它在执行操作后检查结果，而 while 则是执行前进行检查。

```
DELIMITER //
CREATE PROCEDURE proc5 ()
  begin 
    declare v int;
    set v=0;
    repeat
      insert into t values(v);
      set v=v+1;
    until v>=5  end repeat;
 end;
DELIMITER ;
```
3、loop ·····endloop
loop 循环不需要初始条件，这点和 while 循环相似，同时和 repeat 循环一样不需要结束条件, leave 语句的意义是离开循环。
```
DELIMITER //
CREATE PROCEDURE proc6 ()
begin
declare v int;
set v=0;
LOOP_LABLE:loop
   insert into t values(v);
   set v=v+1;
   if v >=5 then
     leave LOOP_LABLE;
   end if;
end loop;
end;
//
DELIMITER ;
```
 LABLES 标号：
标号可以用在 begin repeat while 或者 loop 语句前，语句标号只能在合法的语句前面使用。可以跳出循环，使运行指令达到复合语句的最后一步。
###  ITERATE迭代
ITERATE 通过引用复合语句的标号,来从新开始复合语句:
```
DELIMITER //
CREATE PROCEDURE proc10 ()
begin
  declare v int;
  set v=0;
  LOOP_LABLE:loop
    if v=3 then 
       set v=v+1;
       ITERATE LOOP_LABLE;
    end if;
    insert into t values(v);
    set v=v+1;
    if v>=5 then
       leave LOOP_LABLE;
    end if;
  end loop;
end;
//
mysql > DELIMITER ;
```
## 游标（cursor）的使用
在 MySQL 中，存储过程或函数中的查询有时会返回多条记录，而使用简单的 SELECT 语句，没有办法得到第一行、下一行或前十行的数据，这时可以使用游标来逐条读取查询结果集中的记录。游标在部分资料中也被称为光标。
关系数据库管理系统实质是面向集合的，在 MySQL 中并没有一种描述表中单一记录的表达形式，除非使用 WHERE 子句来限制只有一条记录被选中。所以有时我们必须借助于游标来进行单条记录的数据处理。
一般通过游标定位到结果集的某一行进行数据修改。
个人理解游标就是一个标识，用来标识数据取到了什么地方，如果你了解编程语言，可以把他理解成数组中的下标。
### 声明游标
MySQL 中使用 DECLARE 关键字来声明游标，并定义相应的 SELECT 语句，根据需要添加 WHERE 和其它子句。其语法的基本形式如下：

```javascript
DECLARE cursor_name CURSOR FOR select_statement;
```

其中，cursor_name 表示游标的名称；select_statement 表示 SELECT 语句，可以返回一行或多行数据。
###  打开游标
声明游标之后，要想从游标中提取数据，必须首先打开游标。在 MySQL 中，打开游标通过 OPEN 关键字来实现，其语法格式如下：

```sql
OPEN cursor_name;
```

其中，cursor_name 表示所要打开游标的名称。需要注意的是，打开一个游标时，游标并不指向第一条记录，而是指向第一条记录的前边。
在程序中，一个游标可以打开多次。用户打开游标后，其他用户或程序可能正在更新数据表，所以有时会导致用户每次打开游标后，显示的结果都不同。
### 使用游标
游标顺利打开后，可以使用 FETCH...INTO 语句来读取数据，其语法形式如下：
FETCH cursor_name INTO var_name [,var_name]...
上述语句中，将游标 cursor_name 中 SELECT 语句的执行结果保存到变量参数 var_name 中。变量参数 var_name 必须在游标使用之前定义。使用游标类似高级语言中的数组遍历，当第一次使用游标时，此时游标指向结果集的第一条记录。
MySQL 的游标是只读的，也就是说，你只能顺序地从开始往后读取结果集，不能从后往前，也不能直接跳到中间的记录。
### 关闭游标
游标使用完毕后，要及时关闭，在 MySQL 中，使用 CLOSE 关键字关闭游标，其语法格式如下：

```sql
CLOSE cursor_name;
```

CLOSE 释放游标使用的所有内部内存和资源，因此每个游标不再需要时都应该关闭。
在一个游标关闭后，如果没有重新打开，则不能使用它。但是，使用声明过的游标不需要再次声明，用 OPEN 语句打开它就可以了。
如果你不明确关闭游标，MySQL 将会在到达 END 语句时自动关闭它。游标关闭之后，不能使用 FETCH 来使用该游标。
### 注意
当使用MySQL游标时，还必须声明一个NOT FOUND处理程序来处理当游标找不到任何行时的情况。 因为每次调用FETCH语句时，游标会尝试读取结果集中的下一行。 当光标到达结果集的末尾时，它将无法获得数据，并且会产生一个条件。 处理程序用于处理这种情况。
要声明一个NOT FOUND处理程序，参考以下语法：

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND SET finished = 1;
```

finished是一个变量，指示光标到达结果集的结尾。请注意，处理程序声明必须出现在存储过程中的变量和游标声明之后。
### 实例
声明
```
DECLARE finished INTEGER DEFAULT 0;
DECLARE email varchar(255) DEFAULT "";

-- declare cursor for employee email
DEClARE email_cursor CURSOR FOR 
 SELECT email FROM employees;

-- declare NOT FOUND handler
DECLARE CONTINUE HANDLER 
FOR NOT FOUND SET finished = 1;
```
打开
```
OPEN email_cursor;
```
迭代，在循环中，使用v_finished变量来检查列表中是否有任何电子邮件来终止循环。
```
get_email: LOOP
 FETCH email_cursor INTO v_email;
 IF v_finished = 1 THEN 
 LEAVE get_email;
 END IF;
 -- build email list
 SET email_list = CONCAT(v_email,";",email_list);
END LOOP get_email;
```
可以使用以下脚本测试build_email_list存储过程：
SET @email_list = "";
CALL build_email_list(@email_list);
SELECT @email_list;
## 调试方法
追踪存储过程执行步骤
mysql不像oracle有plsqldevelper工具用来调试存储过程，所以有两简单的方式追踪执行过程：
用一张临时表，记录调试过程
直接在存储过程中，增加select @xxx,在控制台查看结果：
