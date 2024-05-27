https://www.cainiaoya.com/elasticsearch/elasticsearch-search-api.html
# 第一章

## 关键概念：

**节点(node)**

它指的是单个正在运行的**Elasticsearch**实例。单个物理和虚拟服务器可容纳多个节点，具体取决于它们的物理资源（例如RAM(内存)，存储（硬盘）和处理能力）的能力。  

**集群(Cluster)**

它是一个或多个节点的集合。群集为所有数据提供了跨所有节点的集体索引和搜索功能。  

**索引(index)**

它是不同类型的文档及其属性的集合。索引还使用分片的概念来提高性能。例如，一组文档包含社交网络应用程序的数据。  

**文档（Document）**

它是以JSON格式定义的特定方式的字段集合。每个文档都属于一种类型，并且位于索引内。每个文档都与称为UID的唯一标识符相关联。  

**扇区(Shard)**

索引在水平方向上细分为碎片。这意味着每个扇区都包含文档的所有属性，但所包含的JSON对象的数量要少于索引。水平分隔使分片成为一个独立的节点，可以将其存储在任何节点中。主分片是索引的原始水平部分，然后将这些主分片复制到副本分片中。  

**副本(Replicas)**

**Elasticsearch**允许用户创建其索引和扇区的副本。副本不仅有助于在发生故障的情况下提高数据的可用性，而且还可以通过在这些副本中执行并行搜索操作来提高搜索性能。  

## 基本使用

学习如何向Elasticsearch添加一些索引(index)，映射和数据

### 索引（index）与文档（document）

#### 新增索引

```
PUT /school
```

响应

```
200 — OK (610 ms)
{
  "index": "school",
  "acknowledged": true,
  "shards_acknowledged": true
}
```

#### 新增数据

```
POST /school/_doc/10
{
   "name":"Saint Paul School", 
   "description":"ICSE Afiliation",
   "street":"Dawarka", 
   "city":"Delhi", 
   "state":"Delhi", 
   "zip":"110075",
   "location":[28.5733056, 77.0122136], 
   "fees":5000,
   "tags":["Good Faculty", "Great Sports"], 
   "rating":"4.5"
}
```

新增之后查询数据

#### 查询数据

```
GET /school/_doc/10
```

响应

```
{
  "_seq_no": 0,
  "_index": "school",
  "_source": {
    "city": "Delhi",
    "rating": "4.5",
    "name": "Saint Paul School",
    "zip": "110075",
    "tags": [
      "Good Faculty",
      "Great Sports"
    ],
    "state": "Delhi",
    "street": "Dawarka",
    "location": [
      28.5733056,
      77.0122136
    ],
    "fees": 5000,
    "description": "ICSE Afiliation"
  },
  "_version": 1,
  "_primary_term": 1,
  "found": true,
  "_id": "10"
}
```

#### 删除数据

您可以通过向Elasticsearch发送HTTP DELETE请求来删除指定的索引，映射或文档。

```
DELETE /school/_doc/10
```

响应

```
{
  "_seq_no": 6,
  "_shards": {
    "successful": 2,
    "failed": 0,
    "total": 2
  },
  "_index": "school",
  "_version": 3,
  "_primary_term": 1,
  "result": "deleted",
  "_id": "10"
}
```

#### 更新数据

```
POST  school/_update/10
{
   "script" : {
      "source": "ctx._source.name = params.sname",
      "lang": "painless",
      "params" : {
         "sname" : "City Wise School"
      }
   }
 }
```

```
{
  "_seq_no": 8,
  "_shards": {
    "successful": 2,
    "failed": 0,
    "total": 2
  },
  "_index": "school",
  "_version": 2,
  "_primary_term": 1,
  "result": "updated",
  "_id": "10"
}
```



## 版本控制

Elasticsearch还提供了版本控制工具。我们可以使用版本查询参数来指定指定文档的版本。

```
PUT school/_doc/5?version=7&version_type=external
{
   "name":"Central School", "description":"CBSE Affiliation", "street":"Nagan",
   "city":"paprola", "state":"HP", "zip":"176115", "location":[31.8955385, 76.8380405],
   "fees":2200, "tags":["Senior Secondary", "beautiful campus"], "rating":"3.3"
}
```

## 搜索API



