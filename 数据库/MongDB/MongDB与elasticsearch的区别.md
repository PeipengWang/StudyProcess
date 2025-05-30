# MongoDB和Elasticsearch的对比

# MongoDB vs Elasticsearch

|                | MongoDB              | ElasticSearch      | 备注                                                         |
| -------------- | -------------------- | ------------------ | ------------------------------------------------------------ |
| 定位           | (文档型)数据库       | (文档型)搜索引擎   | 一个管理数据,一个检索数据                                    |
| 资源占用       | 一般                 | 高                 | mongo使用c++, es使用Java开发                                 |
| 写入延迟       | 低                   | 高                 | es的写入延迟默认1s, 可配置, 但是要牺牲一些东西               |
| 全文索引支持度 | 一般                 | 非常好             | es本来就是搜索引擎, 这个没啥可比性                           |
| 有无Schema     | 无                   | 无                 | 两者都是无Schema                                             |
| 支持的数据量   | PB+                  | TB+ ~ PB           | 两者支持的量并不好说的太死, 都支持分片和横向扩展, 但是相对来说MongoDB的数据量支持要更大一点 |
| 性能           | 非常好               | 好                 | MongoDB在大部分场景性能比es强的多                            |
| 索引结构       | B树                  | LSM树              | es追求写入吞吐量, MongoDB读写比较均衡                        |
| 操作接口       | TCP                  | Restful(Http)      |                                                              |
| 是否支持分片   | 是                   | 是                 |                                                              |
| 是否支持副本   | 是                   | 是                 |                                                              |
| 选主算法       | Bully(霸凌)          | Bully(霸凌)        | 相比于Paxos和Raft算法实现更简单并有一定可靠性上的妥协，但是选举速度比较快 |
| 扩展难度       | 容易                 | 非常容易           | es真的是我用过的扩展最方便的存储系统之一                     |
| 配置难度       | 难                   | 非常容易           |                                                              |
| 地理位置       | 支持                 | 支持               |                                                              |
| 运维工具       | 丰富                 | 一般               |                                                              |
| 插件和引擎     | 有多个存储引擎供选择 | 有大量插件可以使用 | -                                                            |

# 两者的定位

`MongoDB`和`Elasticsearch`都属于NoSQL大家族, 且都属于文档型数据存储

所以这两者的很多功能和特性高度重合, 但其实两者定位完全不同

MongoDB 是 **文档型数据库**, 提供 **数据存储和管理服务**
Elasticsearch 是**搜索服务**, 提供 **数据检索服务**

两者的很大区别在于源数据的存储和管理

- MongoDB作为一个数据库产品, 是拥有源数据管理能力的
- Elasticsearch作为一个搜索引擎, 定位是**提供数据检索服务**, 也就是说我只管查, 不管写 ^_^, Elasticsearch的Mapping不可变也是为此服务的, 带来的代价就是` es不适合作为数据管理者`, es可以从其他数据源同步数据过来提供查询, 但是不适合自己对数据进行存储和管理

es更侧重数据的查询, 各种复杂的花式查询支持的很好, 相比来说 MongoDB的查询能力就显得比较平庸了

由此可见, 对于个人, 如果你有一批数据要看, 但是不经常进行修改, 这个时候毫无疑问可以用es, 但是如果你还打算继续修改数据, 最好就是使用MongoDB，但其实对大多数人公司来讲，这两者的数据管理能力并没有多大的影响

> ps: es修改Mapping的代价非常高, 所以我们一般都是把新数据重新写入一份新索引，然后直接切换读取的别名到新的索引

# 两者读写数据的异同

`MongoDB`和`ElasticSearch`都支持全文索引, 虽然MongoDB的全文索引效果完全无法跟es相比(es毕竟是专业的搜索引擎产品, 着重提供数据的检所支持, 这方面吊打MongoDB也是可以理解的)

MongoDB虽然在支持的部分查询功能上稍微弱于es, 但是在大部分场景下性能方面完爆es, 不管是读性能, 还是写性能

es的写入延迟默认为1s, 这个虽然是写入延迟的范畴, 但是毫无疑问是一大缺点, 虽然可以配置为更短的时间, 但是这样就要牺牲一定的数据吞吐量, 会造成更频繁的磁盘刷新操作

es底层使用`Lucene`作为核心引擎, 很多es的设计就是为了匹配Lucene中的概念, 其实es可以看成一个lucene的proxy层包装,将lucene的原生接口封装的更好用, 同时还实现了很多管理和监控等辅助功能, 但是整体来说es上层的模块和lucene的隔阂还是挺明显的, 耦合度上有一定的欠缺

MongoDB则是完整的一个单体数据库产品, 虽然内部的存储引擎也是可插拔式的, 整体而言还是更加的浑然一体

> MongoDB支持多种存储引擎, 本文所有涉及mongo存储引擎的只谈默认的WiredTiger引擎, 其实还有某些方面更优秀的其他引擎,例如: MongoRocks等

# 部署和资源占用

单机部署的话其实MongoDB和Elasticsearch都十分的方便, 不过es相对来说资源占用更多一点, 性能也比MongoDB要弱一点

集群化的部署, 我们一般都会选择分片+副本的部署方式, 这种方式下, es部署起来比MongoDB方便太多, MongoDB要部署一套完整的分片 + 副本模式还是比较麻烦的, 没有经验的人部署起来需要一定的学习成本

资源占用方面, MongoDB可以支持存储文件类型的数据, 作为数据库也有数据压缩能力, es则因为大量的索引存在需要占用大量的磁盘和内存空间

# 可用性和容错

MongoDB和ElasticSearch作为天生分布式的代表产品都支持数据分片和副本

两者都通过分片支持水平扩展, 同时都通过副本来支持高可用(HA)

分片就是一个数据集的数据分为多份, 同时分布在多个节点上存储和管理, 主流分片方式有两种: hash分片和range分片, 两种分片方式各有优势, 适合不同的场景

副本就是一份数据集同时有一个或者多个复制品(有些地方叫主从), 每份复制品都一模一样, 但是为了保证数据的一致性, 往往多个副本中只有一个作为Primary副本(通过选主算法从多个副本中选出Primary), 提供写服务, 其他副本只提供读, 或者只提供备份服务

> ps:es和MongoDB都可以通过副本增强读能力, 这与kafka很不一样(kafka的副本只有备份功能)

## 两者分布式方案的一些不同

MongoDB和Elasticsearch虽然都是分布式服务, 但是还是有一些不同方案的选择的

- 分片和副本单位的划分

MongoDB是以节点为单位划分角色, 一旦一个节点被指定为副本, 其上面的数据都是副本

Elasticsearch是以分片为单位划分角色, 一个节点上即可以拥有某分片的主分片和可以同时拥有另一个分片的副本分片, 同时es还支持自动的副本负载均衡, 如果一个新节点上面什么数据都没有, 系统会自动分配分片数据过来

- 架构模式

MongoDB的副本和分片是两种不同的模式, 虽然可以同时使用但是依然有各自的架构设计, 用户可以任意选择选型进行搭配, 每个节点的职责更加专一, 方便据此调整机器配置和进行优化

Elasticsearch中的分片 + 副本是一套统一的架构设计, 每个节点具有接近同等的地位, 配置使用起来更加简单, 但是如果要针对节点所负责的功能对机器进一步做定制就不如MongoDB灵活

# 文档型数据库的特点和问题

## 无schema

文档型数据存储既能享受无schema限制带来的灵活, 又能享受索引查询的快速和类SQL查询的便捷

使他们用起来不像传统的RDBMS那么麻烦, 又不像 Redis,Hbase这种数据库查询功能不够强大, 处在一个传统RDBMS和经典K-V存储之间的比较均衡的位置

我个人很喜欢这个特性, 没有schema的限制, 存储数据更方便也更灵活了, 但是有得有失, 很多固定schema的好处就无法享受到了, 比如: 对数据的高效压缩

## 鸡肋的Collection 和 Type

早期为了跟传统rdbms数据库保持概念一致 ，mongodb和elasticsearch都设计了跟传统数据库里面的`库->表->记录行`对应的概念，具体如下

| RDBMS | MongoDB | Elasticsearch |
| ----- | ------- | ------------- |
| 库    | 库      | 索引          |
| 表    | 集合    | 类型          |
| 记录  | 文档    | 文档          |

其实对于nosql数据库来讲, 集合/类型的意义其实不大, Nosql数据库几乎都是k-v类型的存储结构，完全可以通过key进行业务隔离和区分，真的没有必要为了跟传统数据库对应强行搞出来一个中间概念 ^_^

Elasticsearch从`6.x`版本开始强制只允许一个索引使用一个type, 其实就是意识到这个这个设计的失误, 不想让你用这个type类型, 因为type和传统数据库里面的表概念其实是不一样的，这种概念类比给人造成了误解，到了es的7.x版本会默认取消type类型, 就说明这个type字段真的是鸡肋的不行

## 弱事务

MongoDB以前只是支持同一文档内的原子更新, 以此来实现伪事务功能, 不过Mongo4.0支持Replica Set事务, 大大加强了事务方面的能力

es在这方面倒没有什么进展，因为从应用场景上es对事务的需求不高，不过用户其实也可以使用同文档更新或者通过程序自己来实现事务机制

## 无join支持

文档型数据库大多数都不支持join(也有少量支持的), 但是我一般也用不上多表join的功能, 即便真的需要使用join也可以通过应用层或者通过耦合数据来实现（不过据说未来Mongo4.2版本会带来对join的支持）

不支持join带来的问题就是我们需要自己对数据进行连接, 但是这在擅长使用分布式计算的大数据领域不算什么问题, 相应的缺少join功能可能对善于使用SQL的数据分析师就不大友好

## Bully的选主算法的缺陷

elasticsearch和MongoDB选择的选主算法实现很简单, 但是代价就是有几率出现脑裂的情况, 当然, 具体情况跟配置也有关系(比如:你有三个es节点但是设置的最小主节点数为1, 将最小主节点数设置为2可以避免脑裂情况)

不过脑裂问题一方面发生概率较低，另一方面即使出现了脑裂的情况, 使用`重启大法`一般就能解决 ^_^

总体来说, 这方面不如使用Paxos和Raft算法或者使用zk做协调器的其他分布式系统靠谱

# 其他

- 运维工具

两者背后都有商业公司的支持

MongoDB的很多客户端和运维工具更丰富, 但是MongoDB作为一个数据库产品, 相对应的对运维人员的要求也要更高一点

Elasticsearch则有整套的数据分析和收集工具提供, 配套的kibana就是一个很不错的管控es的工具

- 操作接口

es使用Restful来提供统一的操作接口, 屏蔽了各种语言之间的障碍, 但是同样带来了表达能力和性能的损失

MongoDB则使用TCP, 降低了序列化和网络这一层的性能损耗, 并最大程度保留了接口的内容表达能力, 但是相对的使用起来就不如http那么的方便

# 适用场景

两者其实在很多使用场景上有重合之处, 是可以互相替代, 比如日志收集

但是某些方面两者又各有特色，比如： 如果打算使用一个文档型的业务数据库， 那最好还是选mongodb, 如果你有要求复杂查询又并发性能要求高的场景，类似搜索服务，那最好的选择是elasticsearch

除此之外：

MongoDB有多个存储引擎可以选择, 而且MongoDB不仅看重数据的分析, 对数据的管理同样看重, 总的来说MongoDB更倾向于数据的存储和管理, 可以作为数据源对外提供， 未来说不定还会有支持join和支持倒排索引的mongo引擎出现

Elasticsearch则有很多插件可以使用, 相对来讲Elasticsearch更倾向于数据的查询, 一般情况下elasticsearch仅作为数据检索服务和数据分析平台, 不直接作为源数据管理者

- MongoDB适合

1. 对服务可用性和一致性有高要求
2. 无schema的数据存储 + 需要索引数据
3. 高读写性能要求, 数据使用场景简单的海量数据场景
4. 有热点数据, 有数据分片需求的数据存储
5. 日志, html, 爬虫数据等半结构化或图片，视频等非结构化数据的存储
6. 有js使用经验的人员(MongoDB内置操作语言为js)

- Elasticsearch适合

1. 已经有其他系统负责数据管理
2. 对复杂场景下的查询需求，对查询性能有要求, 对写入及时性要求不高的场景
3. 监控信息/日志信息检索
4. 小团队但是有多语言服务，es拥有restful接口，用起来最方便

# 总结

MongoDB和Elasticsearch都是我比较喜欢的存储产品

两者的功能特性也存在很多重合的地方, 其实现在很多数据库产品都在互相借(chao)鉴(xi), 功能和特性都在逐渐变得相似, 这也是未来很多存储产品的发展趋势, 大家都希望自己能覆盖尽量多的场景和用户群体

很多产品总是在不断的从`没有`->`有`->`功能丰富`,但是功能丰富一定是做了很多的妥协, 于是又有了 `功能众多的单体服务`->`多个功能单一的子服务` 方向的转变,就像三国里面说的 “天下大势, 分久必合合久必分”.

现在NoSQL数据库产品就在这个路上, NoSQL归根到底都是 RDBMS的某个方面的妥协, 现在各种NoSQL 也都在加入对经典SQL和传统RDBMS的 join, 事务的支持, 但是我相信等到两者区别足够小的时候, 一定会有放弃了大而全, 而专注于某一场景的新的存储产品出现，到时候搞不好又是一波新的Nosql潮流