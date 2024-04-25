# CAP与BASE理论
## CAP
一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）这三项中的两项。
C一致性：状态的一致性，缓存，数据库，集群等
A可用性：要求系统内的节点们接收到了无论是写请求还是读请求，都要能处理并给回响应结果
P分区容错性：在分布式系统中，节点通信出现了问题，那么就出现了分区。:能容忍网络分区，在网络断开的情况下，被分隔的节点仍能正常对外提供服务。
一般来说使用网络通信的分布式系统，无法舍弃P性质，那么就只能在一致性和可用性上做一个艰难的选择。
## BASE
BASE是“Basically Available, Soft state, Eventually consistent(基本可用、软状态、最终一致性)”的首字母缩写。其中的软状态和最终一致性这两种技巧擅于对付存在分区的场合，并因此提高了可用性。
Basically Available（基本可用）分布式系统在出现不可预知故障的时候，允许损失部分可用性Soft state（软状态）软状态也称为弱状态，和硬状态相对，是指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同节点的数据副本之间进行数据同步的过程存在延时。Eventually consistent（最终一致性）最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性
## CAP与BASE的比较
CAP 与 BASE 关系BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的结论，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性（Strong consistency），更具体地说，是对 CAP 中 AP 方案的一个补充。其基本思路就是：通过业务，牺牲强一致性而获得可用性，并允许数据在一段时间内是不一致的，但是最终达到一致性状态。

## 参考
https://cloud.tencent.com/developer/article/1860632
https://pdai.tech/md/dev-spec/spec/dev-th-cap.html
https://pdai.tech/md/dev-spec/spec/dev-th-base.html

