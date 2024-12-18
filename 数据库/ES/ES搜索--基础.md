# 搜索

## 空搜索

搜索API的最基础的形式是没有指定任何查询的空搜索，它简单地返回集群中所有索引下的所有文档：

```
{
   "hits" : {
      "total" :       14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       },
        ... 9 RESULTS REMOVED ...
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}
```

### hits

返回结果中最重要的部分是 `hits` ，它包含 `total` 字段来表示匹配到的文档总数，并且一个 `hits` 数组包含所查询结果的前十个文档。

在 `hits` 数组中每个结果包含文档的 `_index` 、 `_type` 、 `_id` ，加上 `_source` 字段。这意味着我们可以直接从返回的搜索结果中使用整个文档。这不像其他的搜索引擎，仅仅返回文档的ID，需要你单独去获取文档。

每个结果还有一个 `_score` ，它衡量了文档与查询的匹配程度。默认情况下，首先返回最相关的文档结果，就是说，返回的文档是按照 `_score` 降序排列的。在这个例子中，我们没有指定任何查询，故所有的文档具有相同的相关性，因此对所有的结果而言 `1` 是中性的 `_score` 。

`max_score` 值是与查询所匹配文档的 `_score` 的最大值。

###  took

`took` 值告诉我们执行整个搜索请求耗费了多少毫秒。

### shards

`_shards` 部分告诉我们在查询中参与分片的总数，以及这些分片成功了多少个失败了多少个。正常情况下我们不希望分片失败，但是分片失败是可能发生的。如果我们遭遇到一种灾难级别的故障，在这个故障中丢失了相同分片的原始数据和副本，那么对这个分片将没有可用副本来对搜索请求作出响应。假若这样，Elasticsearch 将报告这个分片是失败的，但是会继续返回剩余分片的结果。

### timeout

`timed_out` 值告诉我们查询是否超时。默认情况下，搜索请求不会超时。如果低响应时间比完成结果更重要，你可以指定 `timeout` 为 10 或者 10ms（10毫秒），或者 1s（1秒）：

```
GET /_search?timeout=10ms
```

## 多索引查询

**`/_search`**

在所有的索引中搜索所有的类型

- **`/gb/_search`**

  在 `gb` 索引中搜索所有的类型

- **`/gb,us/_search`**

  在 `gb` 和 `us` 索引中搜索所有的文档

- **`/g\*,u\*/_search`**

  在任何以 `g` 或者 `u` 开头的索引中搜索所有的类型

- **`/gb/user/_search`**

  在 `gb` 索引中搜索 `user` 类型

- **`/gb,us/user,tweet/_search`**

  在 `gb` 和 `us` 索引中搜索 `user` 和 `tweet` 类型

- **`/_all/user,tweet/_search`**

  在所有的索引中搜索 `user` 和 `tweet` 类型

当在单一的索引下进行搜索的时候，Elasticsearch 转发请求到索引的每个分片中，可以是主分片也可以是副本分片，然后从每个分片中收集结果。多索引搜索恰好也是用相同的方式工作的—只是会涉及到更多的分片。

## 分页

和 SQL 使用 `LIMIT` 关键字返回单个 `page` 结果的方法相同，Elasticsearch 接受 `from` 和 `size` 参数：

- **`size`**

  显示应该返回的结果数量，默认是 `10`

- **`from`**

  显示应该跳过的初始结果数量，默认是 `0`

如果每页展示 5 条结果，可以用下面方式请求得到 1 到 3 页的结果：

```
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

考虑到分页过深以及一次请求太多结果的情况，结果集在返回之前先进行排序。 但请记住一个请求经常跨越多个分片，每个分片都产生自己的排序结果，这些结果需要进行集中排序以保证整体顺序是正确的。

**在分布式系统中深度分页**

理解为什么深度分页是有问题的，我们可以假设在一个有 5 个主分片的索引中搜索。 当我们请求结果的第一页（结果从 1 到 10 ），每一个分片产生前 10 的结果，并且返回给 *协调节点* ，协调节点对 50 个结果排序得到全部结果的前 10 个。

现在假设我们请求第 1000 页—结果从 10001 到 10010 。所有都以相同的方式工作除了每个分片不得不产生前10010个结果以外。 然后协调节点对全部 50050 个结果排序最后丢弃掉这些结果中的 50040 个结果。

可以看到，在分布式系统中，对结果排序的成本随分页的深度成指数上升。这就是 web 搜索引擎对任何查询都不要返回超过 1000 个结果的原因。

## 轻量搜索

有两种形式的 `搜索` API：一种是 “轻量的” *查询字符串* 版本，要求在查询字符串中传递所有的参数，另一种是更完整的 *请求体* 版本，要求使用 JSON 格式和更丰富的查询表达式作为搜索语言。

查询字符串搜索非常适用于通过命令行做即席查询。例如，查询在  `tweet` 字段包含 `elasticsearch` 单词的所有文档：

```
GET /*/_search?q=tweet:elasticsearch
```

下一个查询在 `name` 字段中包含 `john` 并且在 `tweet` 字段中包含 `mary` 的文档。实际的查询就是这样

```
+name:john +tweet:mary
```

但是查询字符串参数所需要的 *百分比编码* （译者注：URL编码）实际上更加难懂：

```
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```

在 Elasticsearch 7.x+ 版本中，`_all` 已经被移除，不再支持对所有索引进行搜索。你需要替换 `_all` 为具体的索引名称或者使用通配符 `*` 来匹配所有索引。

```
GET /*/_search?q=tweet:elasticsearch AND name:john AND tweet:mary
```

##  更复杂的查询

下面的查询针对tweents类型，并使用以下的条件：

- `name` 字段中包含 `mary` 或者 `john`

- `date` 值大于 `2014-09-10`

- `_all` 字段包含 `aggregations` 或者 `geo`

  ```
  GET /_all/_search?q=(name:mary OR name:john) AND date:>2014-09-10 AND (field1:aggregations OR field1:geo OR field2:aggregations OR field2:geo)
  ```

  使用 **AND** 表示必须匹配的条件。

  使用 **OR** 表示其中一个条件匹配即可。

  使用 `>`、`<`、`>=`、`<=` 等符号进行范围比较。

  使用括号 `()` 对条件进行分组，确保逻辑清晰。

  指定字段：`field_name:query_value`。

# 请求体查询

## 空查询

以最简单的 `search` API 的形式开启我们的旅程，空查询将返回所有索引库（indices)中的所有文档：

```
GET /_search
{}
```

分页查询

```
POST 索引/_search
{
  "from": 2,
  "size": 10
}
```

## 查询表达式

查询表达式(Query DSL)是一种非常灵活又富有表现力的 查询语言。 Elasticsearch 使用它可以以简单的 JSON 接口来展现 Lucene 功能的绝大部分。在你的应用中，你应该用它来编写你的查询语句。它可以使你的查询语句更灵活、更精确、易读和易调试。

要使用这种查询表达式，只需将查询语句传递给 `query` 参数：

```
POST 索引/_search
{
    "query": {
      "match_all": {}
    }
}
```

###  查询语句的结构

一个查询语句的典型结构：

```js
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```

如果是针对某个字段，那么它的结构如下

```
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

举个例子，你可以使用 `match` 查询语句 来查询 `tweet` 字段中包含 `elasticsearch` 的 tweet：

```
{
    "match": {
        "tweet": "elasticsearch"
    }
}
```

完整的查询请求如下，查询索引中tweet字段中包含elasticsearch的文档

```
GET 索引/_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

### 合并查询语句

#### **bool 查询简介**

在 Elasticsearch 中，`bool` 查询是一种复合查询，它允许将多个查询条件组合起来，并通过逻辑运算（`must`、`must_not`、`should` 和 `filter`）进行灵活筛选。

**用途**：
`bool` 查询用于构建更复杂的查询逻辑，类似于 SQL 中的 `AND`、`OR` 和 `NOT` 逻辑。

#### ES 查询的逻辑规则

- **must**: 等同于 `AND`，必须满足。
- **must_not**: 等同于 `NOT`，必须不满足。
- **should**: 等同于 `OR`，可以满足，但满足时分数更高。
- **filter**: 不影响分数，仅做过滤。

```
POST */_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "id": "00D4D20" }},
        { 
          "range": {
            "time": {
              "gte": "2024-12-17T07:43:48.833",
              "lte": "2024-12-18T07:43:47.833"
            }
          }
        }
      ]
    }
  },
  "from": 10,
  "size": 20
}

```

## 最重要的查询

### match_all 查询

`match_all` 查询简单的匹配所有文档。在没有指定查询方式时，它是默认的查询：

```sense
{ "match_all": {}}
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/70_Match_all_query.json) 

它经常与 filter 结合使用—例如，检索收件箱里的所有邮件。所有邮件被认为具有相同的相关性，所以都将获得分值为 `1` 的中性 `_score`。

### match 查询

无论你在任何字段上进行的是全文搜索还是精确查询，`match` 查询是你可用的标准查询。

如果你在一个全文字段上使用 `match` 查询，在执行查询前，它将用正确的分析器去分析查询字符串：

```sense
{ "match": { "tweet": "About Search" }}
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/70_Match_query.json) 

如果在一个精确值的字段上使用它，例如数字、日期、布尔或者一个 `not_analyzed` 字符串字段，那么它将会精确匹配给定的值：

```sense
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/70_Match_query.json) 

对于精确值的查询，你可能需要使用 filter 语句来取代 query，因为 filter 将会被缓存。接下来，我们将看到一些关于 filter 的例子。

不像我们在 [*轻量* 搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-lite.html) 章节介绍的字符串查询（query-string search）， `match` 查询不使用类似 `+user_id:2 +tweet:search` 的查询语法。它只是去查找给定的单词。这就意味着将查询字段暴露给你的用户是安全的；你需要控制那些允许被查询字段，不易于抛出语法异常。





##  multi_match 查询

`multi_match` 查询可以在多个字段上执行相同的 `match` 查询：

```sense
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```





###  range 查询

`range` 查询找出那些落在指定区间内的数字或者时间：

```sense
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/70_Range_filter.json) 

被允许的操作符如下：

- **`gt`**

  大于

- **`gte`**

  大于等于

- **`lt`**

  小于

- **`lte`**

  小于等于

### term 查询

`term` 查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 `not_analyzed` 的字符串：

```sense
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/70_Term_filter.json) 

`term` 查询对于输入的文本不 *分析* ，所以它将给定的值进行精确查询。

### terms 查询

`terms` 查询和 `term` 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件：

```sense
{ "terms": { "tag": [ "search", "full_text", "nosql" ] }}
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/70_Terms_filter.json) 

和 `term` 查询一样，`terms` 查询对于输入的文本不分析。它查询那些精确匹配的值（包括在大小写、重音、空格等方面的差异）。

### exists 查询和 missing 查询

`exists` 查询和 `missing` 查询被用于查找那些指定字段中有值 (`exists`) 或无值 (`missing`) 的文档。这与SQL中的 `IS_NULL` (`missing`) 和 `NOT IS_NULL` (`exists`) 在本质上具有共性：

```sense
{
    "exists":   {
        "field":    "title"
    }
}
```

拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/054_Query_DSL/70_Exists_filter.json) 

这些查询经常用于某个字段有值的情况和某个字段缺值的情况。
