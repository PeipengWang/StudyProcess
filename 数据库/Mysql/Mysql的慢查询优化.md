# 如何优化慢查询
慢 SQL 的优化，主要从两个方面考虑，SQL 语句本身的优化，以及数据库设计的优化。
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/4134838b3f8d4f4eae470642ed50eb7f.png)


1、避免不必要的列  
覆盖索引会导致回表，且增大了IO  
2、分页优化  
[深分页解决方案](https://blog.csdn.net/m0_71777195/article/details/125600166)
使用子查询in  
使用连接表 left join  
使用游标，只能一页一页的翻  
3、索引优化  
注意索引失效的场景[索引失效](https://blog.csdn.net/Artisan_w/article/details/134833315)  
4、jion优化  
优化子查询  
尽量使用 Join 语句来替代子查询，因为子查询是嵌套查询，而嵌套查询会新创建一张临时表，而临时表的创建与销毁会占用一定的系统资源以及花费一定的时间，同时对于返回结果集比较大的子查询，其对查询性能的影响更大  
小表驱动大表  
适当增加冗余字段  
避免使用 JOIN 关联太多的表  
5、排序优化  
利用索引扫描做排序  
6、union优化  
MySQL 处理 union 的策略是先创建临时表，然后将各个查询结果填充到临时表中最后再来做查询，很多优化策略在 union 查询中都会失效，因为它无法利用索引  
最好手工将 where、limit 等子句下推到 union 的各个子查询中，以便优化器可以充分利用这些条件进行优化  
此外，除非确实需要服务器去重，一定要使用 union all，如果不加 all 关键字，MySQL 会给临时表加上 distinct 选项，这会导致对整个临时表做唯一性检查，代价很高。  
[链接](https://javabetter.cn/sidebar/sanfene/mysql.html#_25-有哪些方式优化慢-sql)