
# MySQL 添加注释(comment)
在MySQL数据库中， 字段或列的注释是用属性comment来添加。

创建新表的脚本中， 可在字段定义脚本中添加comment属性来添加注释。
例代码如下：

create table test(
id int not null default 0 comment '用户id'
)

如果是已经建好的表， 也可以用修改字段的命令，然后加上comment属性定义，就可以添加上注释了。
## 字段的注释
查看已有表的所有字段的注释呢？
可以用命令：show full columns from table 来查看， 示例如下：

```
show full columns from test;
```
修改字段的注释
```
alter table test1 modify column field_name int comment '修改后的字段注释'; 
```
-- 注意：字段名和字段类型照写就行
查看字段注释的方法
```
-- show
show  full  columns  from  test1;
```
-- 在元数据的表里面看
```
select * from COLUMNS where TABLE_SCHEMA='my_db' and TABLE_NAME='test1' \G
```
## 表注释
创建表的时候写注释
```
create table test1 (
    field_name int comment '字段的注释'
)comment='表的注释';
```
修改表的注释
alter table test1 comment '修改后的表的注释';

查看表注释的方法
-- 在生成的SQL语句中看 
show  create  table  test1;
-- 在元数据的表里面看
```
use information_schema;
select * from TABLES where TABLE_SCHEMA='my_db' and TABLE_NAME='test1' \G
```

