group by 与 order by的用法
GROUP BY 语句用于结合聚合函数，根据一个或多个列对结果集进行分组。
SQL GROUP BY 语法
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;

下面是 "access_log" 网站访问记录表的数据：

mysql> SELECT * FROM access_log;
+-----+---------+-------+------------+
| aid | site_id | count | date |
+-----+---------+-------+------------+
| 1 | 1 | 45 | 2016-05-10 |
| 2 | 3 | 100 | 2016-05-13 |
| 3 | 1 | 230 | 2016-05-14 |
| 4 | 2 | 10 | 2016-05-14 |
| 5 | 5 | 205 | 2016-05-14 |
| 6 | 4 | 13 | 2016-05-15 |
| 7 | 3 | 220 | 2016-05-15 |
| 8 | 5 | 545 | 2016-05-16 |
| 9 | 3 | 201 | 2016-05-17 |
+-----+---------+-------+------------+
9 rows in set (0.00 sec)
GROUP BY 简单应用
统计 access_log 各个 site_id 的访问量：
![在这里插入图片描述](https://img-blog.csdnimg.cn/948dbb405a594c5fb976a896865a59bb.png)

实例
SELECT site_id, SUM(access_log.count) AS nums
FROM access_log GROUP BY site_id;

**HAVING 子句**
在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与聚合函数一起使用。
HAVING 子句可以让我们筛选分组后的各组数据。
SQL HAVING 语法

```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value;
```

where 和having之后都是筛选条件，但是有区别的：
1.where在group by前， having在group by 之后
2.聚合函数（avg、sum、max、min、count），不能作为条件放在where之后，但可以放在having之后

SQL ORDER BY 关键字
ORDER BY 关键字用于对结果集按照一个列或者多个列进行排序。
ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 DESC 关键字。

```
SQL ORDER BY 语法
SELECT column_name,column_name
FROM table_name
ORDER BY column_name,column_name ASC|DESC;
```

默认是升序排序的。