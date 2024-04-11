


@[TOC](SpringBoot与检索)

[ElasticSearch中文官方文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/getting-started.html)
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 前言


我们的应用经常需要添加检索功能，开源的 ElasticSearch 是目前全文搜索引擎的首选。他可以快速的存储、搜索和分析海量数据。SpringBoot通过整合Spring Data ElasticSearch为我们提供了非常便捷的检索功能支持；Elasticsearch是一个分布式搜索服务，提供Restful API，底层基于Lucene，采用多shard（分片）的方式保证数据安全，并且提供自动resharding的功能，github等大型的站点也是采用了ElasticSearch作为其搜索服务，

# 一、Linux环境下安装ElasticSearch？

```shell
Trying to pull elasticsearch 
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127112832656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

```shell
 docker pull elasticsearch
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127112920324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
查看下载情况：
如果出现错误可以尝试加上版本号
```
docker images
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127113152295.png)
elsaticsearch默认用java写的，会默认占用2G的内存空间，因此在启动的时候可能需要对内存进行限制，如下所示：

```shell
 docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9300:9300 -p 9200:9200 --name ES02 5acf0e8da90b
```
限制为256m，并且端口分别为
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127113721607.png)

访问：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112713472922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
# 二、 概念原理
以 员工文档 的形式存储为例：一个文档代表一个员工数据。存储数据到ElasticSearch 的行为叫做 索引 ，但在索引一个文档之前，需要确定将文档存储在哪里。
 一个 ElasticSearch 集群可以 包含多个 索引 ，相应的每个索引可以包含多个 类型 。 这些不同的类型存储着多个 文档 ，每个文档又有 多个 属性 。
类似关系：
– 索引-数据库
– 类型-表
– 文档-表中的记录
– 属性-列
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127150234375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
发送请求：/索引名称/类型名称/特定属性
# 三、 利用Postman软件测试ES（快速入门）
## 1. 发送数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127151709737.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
## 2. 响应数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127151759812.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

## 3. 查找：利用GET方法
### （1）简单查找
直接利用精确地索引来查找
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201127152153507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
### （2）查找所有

```shell
服务器IP:9200/megacorp/employee/_search
```
利用_search来查找megacorp/employee下的所有数据
### （3）条件查找

```java
服务器IP:9200/megacorp/employee/_search?q=last_name:Smith
```
上面是查找megacorp/employee下的last_name是Simth的数据
### （4）表达式查找

```c
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
          "last_name" : "Smith"
        }
    }
}
```
精确查找last_name为Smith的数据

```c
GET /megacorp/employee/_search
{
    "query" : {
       "must"{
         "match" : {
           "last_name" : "Smith"
          }
       } 
       "fliter":{
         "range":{
           "age" : {"gt" : 30}
         }
       }
    }
}
```
"fliter" 过滤age大于30的Last_name为Smith的数据
还有短语搜索，高亮搜索。。。。详细见官方文档[搜索方式](https://www.elastic.co/guide/cn/elasticsearch/guide/2.x/_finding_exact_values.html)

## 4.  删除：利用DELETE方法

# 四、SpringBoot整合ES
## 1. 导入依赖

```xml
      <!--ElasticSearch-->
         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```
## 2. 配置文件
由于不同版本有很大的不同，需要进一步去学习


 