# 索引
创建索引
```
create index index_name on table_name(column_name)
```
删除索引
```
drop index index_name
```
创建组合索引
```
create index index_name on table_name(column_name_1,column_name_2);
```
查看索引
-- 查看某个表的索引
```
SELECT * from  all_indexes WHERE  table_name=upper('table_name');
```
-- 查看索引在哪个字段

```
select * from user_ind_columns where table_name=upper('table_name');
```

-- 查看某个用户下面所有的索引

```
SELECT * from  all_indexes WHERE owner='SR';
```
查询某个用户下面所有表的数据量
-- 查询某个表的数据量  也就是有多少条数据

```
SELECT t.table_name,t.num_rows from user_tables t ORDER BY t.NUM_ROWS DESC;
```
-- 查询某个用户下面的表

```
SELECT * from all_all_tables WHERE owner='SR' and table_name='TABLE_NAME';
```
查询表的索引

select * from all_indexes where table_name = '表名称';


