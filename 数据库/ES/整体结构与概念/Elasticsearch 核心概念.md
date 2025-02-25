# Elasticsearch 核心概念

```cluster
学好ElasticSearch，必先了解其核心概念、名词和属性
ElasticSearch核心概念：Node、Cluster、Shards、Replicas、Index、Type、Document、Settings、Mapping、Analyzer
```

# 一、核心概念

## 1、节点（node）

```
节点是组成ElasticSearch集群的基本服务单元，集群中的每个运行中的ElasticSearch服务器都可称之为节点。
```



## 2、集群（Cluster）

```
Elasticsearch的集群是由具有相同cluster.name(默认值为elasticsearch)的一个或多个Elasticsearch节点组成的，各个节点协同工
作共享数据。同一个集群内节点的名字不能重复，但集群名称一定要相同。
使用ElasticSearch集群时，需要选择一个合适的名字来代替cluster.name(默认值)。默认值是为了防止一个新启动的节点加入相同网络中
的另一个相同的集群中。
```

**Elasticsearch集群中节点状态：**

- **Green（绿色）**：表示节点运行状态为健康状态。所有主分片和副分片都可以正常工作，集群100%健康。
- **Yellow（黄色）**：表示节点运行状态为预警状态。所有的主分片都可以正常工作，但至少有一个副分片是不能正常工作的。此时集群依然可以正常工作，但集群的高可用性在某种程度上被弱化。
- **Red（红色）**：表示集群无法正常使用。此时集群中至少有一个分片的主分片以及它的全部副分片都不可以正常工作。虽然集群的查询操作还可以进行，但是只能返回部分数据（其他正常分片的数据可以返回），而分配到这个问题分片上的写入请求将会报错，最终导致数据丢失。



## 3、分片（Shards）

```
当索引的数据量太多时，受限于单个节点的内存、磁盘处理能力等，节点无法足够快速地响应客户端的请求，此时需要将一个索引上的数据进行水
平拆分。拆分出来的每个数据部分称之为分片。并且一般情况下，每个分片都会放到不同的服务器上。
ElasticSearch中，默认一个索引创建5个主分片，并分别为每个主分片创建一个副本。
当操作者向设有多个分片的索引中写入数据时，是通过路由来确定具体写入哪个分片中。所以创建分片时需要指定分片的数量，并且分片的数量一
旦确定就不能更改
Elasticsearch 依赖于Lucence,并且ElasticSearch中的分片其实就是Lucene中的索引文件，因此，每个分片必须有一个主分片和零到多
个副本分片。
当操作者执行查询索引操作时，需要在索引对应的多个分片上进行查询。Elasticsearch会把查询发送给每个相关的分片，并汇总各个分片的查
询结果。因此，对于上层应用程序而言，应用程序不知道分片的存在。
```



## 4、备份\副本（Replicas ）

```
副本指对主分片的备份，且这种备份是精确复制模式。
每个主分片都可以有零或者多个副本，主分片和备份分片都可以对外提供数据查询服务。
当构建索引进行写入操作时，首先在主分片上完成数据的索引，然后数据会从主分片分发到备份分片上进行索引。
当主分片不可用时，ElasticSearch会在备份分片中选举一个分片作为主分片，从而避免数据丢失。
```

**备份分片的优缺点：**

- **Advantage**：备份分片即可以提升**ElasticSearch**系统的高可用性能，又可以提升搜索时的并发性能。
- **Disadvantage**: 备份分片数量设置太多，会在写操作时增加数据同步的负担。



## 5、索引（Index）

```
索引由一个或多个分片组成
在使用索引时，需要通过索引名称在集群中进行唯一标识
```



## 6、类别（Type）

```
类别指索引内部的逻辑分区，通过Type的名字在索引内进行唯一标识
在查询时，如果没有该值，则表示需要在整个索引中查询
```



## 7、文档（Document）

```
索引中的每一条数据叫作一个文档，与关系型数据库的使用方式类型，一条文档数据通过 _id 在Type内进行唯一标识 
```



## 8、Settings

```
settings是对集群中索引的定义信息，比如一个索引默认的分片数、副本数等
```



## 9、映射（Mapping）

```
* Mapping表示保存了定义索引中字段（Field）的存储类型、分词方式、是否存储等信息，有点类似于关系型数据库中的表结构信息。
* Mapping可以动态识别
* ElasticSearch会根据数据格式自动识别索引中字段的类型，如无特殊需求，则不需要手动创建Mapping
* 当需要对某些字段添加特殊属性时，就需要手动设置Mapping（如是否分词、是否存储、定义使用其他分词器）
* 一个索引的Mapping一旦创建，若已经存储了数据，就不可以修改
```



## 10、Analyzer

```
Analyzer表示字段分词方式的定义。
一个Analyzer通常由一个Tokenizer和零到多个Filter组成。
默认的标准Analyzer包含一标准的Tokenizer和三个Filter，即Standard Token Filter、Lower Case Token Filter 和 Stop
Token Filter
```



# 二、ElasticSearch Vs 关系型数据库

``` 
虽然ElasticSearch不同于关系型数据库，但是其一些概念还是具有相似性的，初学者可以借鉴关系型数据库的概念，来快速入门
ElasticSearch。
```

![image-20201206000306385](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201206000306385.png)

[上一篇：Elasticsearch 配置](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/ElasticSearch%20%E9%85%8D%E7%BD%AE.md)

[下一篇：Elasticsearch 架构设计及说明](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/Elasticsearch%20%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E5%8F%8A%E8%AF%B4%E6%98%8E.md)