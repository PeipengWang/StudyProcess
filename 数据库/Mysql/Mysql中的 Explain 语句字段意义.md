explain（执行计划）包含的信息十分的丰富，着重关注以下几个字段信息。
执行 explain select * from table_name;
会得出以下结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/750d69293bae404cbcd978505266817f.png)
其中，比较重要的结果如下所示：
①id，select子句或表执行顺序，id相同，从上到下执行，id不同，id值越大，执行优先级越高。
②type，type主要取值及其表示sql的好坏程度（由好到差排序）：system>const>eq_ref>ref>range>index>ALL。保证range，最好到ref。
③key，实际被使用的索引列。
④ref，关联的字段，常量等值查询，显示为const，如果为连接查询，显示关联的字段。
⑤Extra，额外信息，使用优先级Using index>Using filesort（九死一生）>Using temporary（十死无生）。

