# 约束
## 概述
   约束是数据库用来确保数据满足业务规则的手段，不过在真正的企业开发中，除了主键约束这类具有强需求的约束，像外键约束，检查约束更多时候仅仅出现在数据库设计阶段，真实环境却很少应用，更多是放到程序逻辑中去进行处理。这也比较容易理解，约束会一定程度上较低数据库性能，有些规则直接在程序逻辑中处理就可以了，同时，也有可能在面对业务变更或是系统扩展时，数据库约束会使得处理不够方便。不过在我看来，数据库约束是保证数据准确性的最后一道防线，对于设计合理的系统，处于性能考虑数据库约束自然可有可无；不过若是面对关联关系较为复杂的系统，且对系统而言，数据的准确性完整性要高于性能要求，那么这些约束还是有必要的（否则，就会出现各种相对业务规则来说莫名其妙的脏数据，本人可是深有体会的。。）。总之，对于约束的选择无所谓合不合理，需要根据业务系统对于准确性和性能要求的侧重度来决定。
 数据库约束有五种：
     主键约束（PRIMARY KEY）
     唯一性约束（UNIQUE)
     非空约束（NOT NULL)
     外键约束（FOREIGN KEY)
     检查约束（CHECK)
## 主键约束（PRIMARY KEY)
   主键是定位表中单个行的方式，可唯一确定表中的某一行，关系型数据库要求所有表都应该有主键，不过Oracle没有遵循此范例要求，Oracle中的表可以没有主键（这种情况不多见）。关于主键有几个需要注意的点：
键列必须必须具有唯一性，且不能为空，其实主键约束 相当于 UNIQUE+NOT NULL
一个表只允许有一个主键
主键所在列必须具有索引（主键的唯一约束通过索引来实现），如果不存在，将会在索引添加的时候自动创建
     添加主键（约束的添加可在建表时创建，也可如下所示在建表后添加，一般推荐建表后添加，灵活度更高一些，建表时添加某些约束会有限制）
```
 alter table emp add constraint emp_id_pk primary key(id);
```
## 唯一性约束（UNIQUE)
   唯一性约束可作用在单列或多列上，对于这些列或列组合，唯一性约束保证每一行的唯一性。
   UNIQUE需要注意：
   对于UNIQUE约束来讲，索引是必须的。如果不存在，就自动创建一个（UNIQUE的唯一性本质上是通过索引来保证的）
    UNIQUE允许null值，UNIQUE约束的列可存在多个null。这是因为，Unique唯一性通过btree索引来实现，而btree索引中不包含null。当然，这也造成了在where语句中用null值进行过滤会造成全表扫描。
     添加唯一约束
```
SQL> alter table emp add constraint emp_code_uq unique(code);
```
## 非空约束（NOT NULL)
非空约束作用的列也叫强制列。顾名思义，强制键列中必须有值，当然建表时候若使用default关键字指定了默认值，则可不输入。
　　添加非空约束，语法较特别
```
SQL> alter table emp modify ename not null;
```
## 外键约束（FOREIGN KEY）
外键约束定义在具有父子关系的子表中，外键约束使得子表中的列对应父表的主键列，用以维护数据库的完整性。不过出于性能和后期的业务系统的扩展的考虑，很多时候，外键约束仅出现在数据库的设计中，实际会放在业务程序中进行处理。外键约束注意以下几点：
　　外键约束的子表中的列和对应父表中的列数据类型必须相同，列名可以不同
　　对应的父表列必须存在主键约束（PRIMARY KEY）或唯一约束（UNIQUE）
　　外键约束列允许NULL值，对应的行就成了孤行了
　　其实很多时候不使用外键，很多人认为会让删除操作比较麻烦，比如要删除父表中的某条数据，但某个子表中又有对该条数据的引用，这时就会导致删除失败。我们有两种方式来优化这种场景：
　　第一种方式简单粗暴，删除的时候，级联删除掉子表中的所有匹配行，在创建外键时，通过 on delete cascade 子句指定该外键列可级联删除：

```
SQL> alter table emp add constraint emp_deptno_fk foreign key(deptno) references dept (deptno) on delete cascade;
```
　　第二种方式，删除父表中的对应行，会将对应子表中的所有匹配行的外键约束列置为NULL，通过 on delete set null 子句实施：

```
SQL> alter table emp add constraint emp_deptno_fk foreign key(deptno) references dept(deptno) on delete set null;
```
实际上，外键约束列和对应的父表列可以在同一张表中，常见的就是表的业务逻辑含义是一棵树，最简单的例子如下（id为主键id，fid为父id，fid存储对id的引用），这种结构的表根据业务要求可通过Oracle的递归查询来获取这种层级关系
## 检查约束（CHECK)
检查约束可用来实施一些简单的规则，比如列值必须在某个范围内。检查的规则必须是一个结果为true或false 的表达式，比如：
```
SQL> alter table emp add constraint emp_sex_ck check(sex in('男','女'));
```
## 另外两种添加约束的方法
alter方法添加约束比较灵活，但是有些业务可能会对用户权限做限制，如果没有alter any table权限，这个可能无法正常使用，这时候还有另外两种添加约束的方法。
我们可以采用在表内添加外键的方式来解决：
一种是无命名的方式
```
create table jack (id int primary key not null,name varchar2(20));
```
另一种是有命名的方式
```
create table jack (id int ,name varchar2(20),constraint ixd_id primary key(id));
```
这两种方式如果分别查询会得到不同的索引结果，感兴趣可以自行查询
```
select table_name,index_name from user_indexes where table_name='JACK';
```
## 其他操作
查询主键
```
select table_name,index_name from user_indexes where table_name='JACK';
```
```
select a.table_name,a.index_name,b.constraint_name,b.constraint_type,b.status from user_indexes a,user_constraints b where b.table_name='JACK'and a.table_name=b.table_name;
```
```
select owner,constraint_name,table_name,column_name from user_cons_columns where table_name = 'JACK';
```
查看constraint
```
SELECT constraint_name, table_name, r_owner, r_constraint_name
FROM all_constraints
WHERE table_name = '大写表名' and owner = '大写用户';
```
删除主键
```
alter table jack drop constraint SYS_C0011105;
```

