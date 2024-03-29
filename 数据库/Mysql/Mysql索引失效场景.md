# Mysql索引失效
## 联合索引不满足最左匹配原则
在联合索引中，最左侧的字段优先匹配。因此，在创建联合索引时，where子句中使用最频繁的字段放在组合索引的最左侧。
而在查询时，要想让查询条件走索引，则需满足：**最左边的字段要出现在查询条件中。**
## 使用了select *
## 索引列参与运算
## 索引列参使用了函数
## 错误的Like使用
针对like的使用非常频繁，但使用不当往往会导致不走索引。常见的like使用方式有：
方式一：like '%abc'；
方式二：like 'abc%'；
方式三：like '%abc%'；
其中方式一和方式三，由于占位符出现在首部，导致无法走索引。这种情况不做索引的原因很容易理解，索引本身就相当于目录，从左到右逐个排序。而条件的左侧使用了占位符，导致无法按照正常的目录进行匹配，导致索引失效就很正常了。
第五种索引失效情况：模糊查询时（like语句），模糊匹配的占位符位于条件的首部。
## 类型隐式转换
例如：
select * from t_user where id_no = 1002;
id_no字段类型为varchar，但在SQL语句中使用了int类型，导致全表扫描。
## 使用OR操作
在使用or关键字时，切记两个条件都要添加索引，否则会导致索引失效。
查询条件使用or关键字，其中一个字段没有创建索引，则会导致整个查询语句索引失效； or两边为“>”和“<”范围查询时，索引失效。
## 两列做比较
如果两个列数据都有索引，但在查询条件中对两列数据进行了对比操作，则会导致索引失效。
## 不等于比较 select * from t_user where id != 2;
查询条件使用不等进行比较时，需要慎重，普通索引会查询结果集占比较大时索引会失效。

## is not null
select * from t_user where id_no is not null;
查询条件使用is null时正常走索引，使用is not null时，不走索引。
## not in和not exists
查询条件使用not in时，如果是主键则走索引，如果是普通索引，则索引失效。
## order by导致索引失效
当查询条件涉及到order by、limit等条件时，是否走索引情况比较复杂，而且与Mysql版本有关，通常普通索引，如果未使用limit，则不会走索引。order by多个索引字段时，可能不会走索引。其他情况，建议在使用时进行expain验证。
## 参数不同导致索引失效
当Mysql发现通过索引扫描的行记录数超过全表的10%-30%时，优化器可能会放弃走索引，自动变成全表扫描。某些场景下即便强制SQL语句走索引，也同样会失效。
类似的问题，在进行范围查询（比如\>、< 、>=、<=、in等条件）时往往会出现上述情况，而上面提到的临界值根据场景不同也会有所不同。
当查询条件为大于等于、in等范围查询时，根据查询结果占全表数据比例的不同，优化器有可能会放弃索引，进行全表扫描。
## 其他
Mysql优化器的其他优化策略，比如优化器认为在某些情况下，全表扫描比走索引快，则它就会放弃索引。
