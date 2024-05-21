# Zookeeper上

## 1、zookeeper应用场景

zooKeeper是一个经典的分布式数据一致性解决方案，致力于为分布式应用提供一

个高性能、高可用，且具有严格顺序访问控制能力的分布式协调存储服务。

- .维护配置信息
- 分布式锁服务
- 集群管理 TM
- 生成分布式唯一ID P席员

### 1.1 维护配置信息

​	java编程经常会遇到配置项，比如数据库的url、schema、user和password等。通常这些配置项我们会放置在配置文件中，再将配置文件放置在服务器上当需要更改配置项时，需要去服务器上修改对应的配置文件。但是随着分布式系统的兴起，由于许多服务都需要使用到该配置文件，**因此有必须保证该配置服务的高可用性（highavailability）和各台服务器上配置数据的一致性**。通常会将配置文件部署在一个集群上，然而一个集群动辄上千台服务器，此时如果再一台台服务器逐个修改配置文件那将是非常繁琐且危险的的操作，因此就需要一种服务，能够高效快速且可靠地完成配置项的更改等操作，并能够保证各配置项在每台服务器上的数据一致性。

​	zookeeper就可以提供这样一种服务，其使用Zab这种一致性协议来保证一致性。现在有很多开源项目使用zookeeper来维护配置，比如在hbase中，客户端就是连接一个zookeeper，获得必要的hbase集群的配置信息，然后才可以进一步操作。还有在开源的消息队列kafka中，也使用zookeeper来维护broker的信息。在alibaba开源的soa框架dubbo中也广泛的使用zookeeper管理一些配置来实现服务治理。



![image-20240520233049997](https://raw.githubusercontent.com/PeipengWang/picture/master/zk/image-20240520233049997.png)

### 1.2 分布式锁服务

​	一个集群是一个分布式系统，由多台服务器组成。为了提高并发度和可靠性，多台服务器上运行着同一种服务。当多个服务在运行时就需要协调各服务的进度，有时候需要保证当某个服务在进行某个操作时，其他的服务都不能进行该操作，即对该操作进行加锁，如果当前机器挂掉后，释放锁并failover到其他的机器继续执行该服务。

### 1.3 集群管理

​	一个集群有时会因为各种软硬件故障或者网络故障，出现某些服务器挂掉而被移除集群，而某些服务器加入到集群中的情况，zookeeper会将这些服务器加入/移出的情况通知给集群中的其他正常工作的服务器，以及时调整存储和计算等任务的分配和执行等。此外zookeeper还会对故障的服务器做出诊断并尝试修复。

### 1.4 生成分布式唯一**ID**

​	在过去的单库单表型系统中，通常可以使用数据库字段自带的auto_increment 属性来自动为每条记录生成一个唯一的ID。但是分库分表后，就无法在依靠数据库的auto_increment属性来唯一标识一条记录了。此时我们就可以用zookeeper在分布式环境下生成全局唯一ID。做法如下：每次要生成一个新Id时，创建一个持久顺序节点，创建操作返回的节点序号，即为新Id，然后把比自己节点小的删除即可

## 2 zookeeper的数据模型

zookeeper的数据节点可以视为树状结构（或者目录），树中的各节点被称为znode（即zookeepernode），一个znode可以有多个子节点。zookeeper节点在结构上表现为树状；使用路径path来定位某个znode，比如/ns-1/itcast/mysql/schema1/table1，此处ns-1、itcast、mysql、schema1、table1分别是根节点、2级节点、3级节点以及4级节点；其中ns-1是itcast的父节点，itcast是ns-1的子节点，itcast是mysql的父节点，mysql是itcast的子节点，以此类推。

​	znode，兼具文件和目录两种特点。既像文件一样维护着数据、元信息、ACL、时间戳等数据结构，又像目录一样可以作为路径标识的一部分。

![image-20240520233952133](https://raw.githubusercontent.com/PeipengWang/picture/master/zk/image-20240520233952133.png)

**那么如何描述一个znode呢？**

一个znode大体上分为3各部分：

- 节点的数据：即znodedata(节点path,节点data)的关系，就像javamap中(key,value)的关系
- 节点的子节点children
- 节点的状态stat：用来描述当前节点

**节点状态stat的属性**

在zookeepershell中使用get命令查看指定路径节点的data、stat信息：

那么如何描述一个znode呢？一个znode大体上分为3各部分：节点的数据：即znodedata(节点path,节点data)的关系就像是javamap中(key,value)的关系

节点的子节点children

节点的状态stat：用来描述当前节点

的创建、修改记录，包括cZxid、ctime等节点状态**stat**的属性在zookeepershell中使用get命令查看指定路径节点的data、stat信息：属性说明：cZxid：数据节点创建时的事务ID

ctime：数据节点创建时的时间

mZxid：数据节点最后一次更新时的事务ID 

mtime：数据节点最后一次更新时的时间

```
[zk:localhost:2181(CONNECTED)7]get/ns-1/tenantcZxid=0x6a0000000a
ctime=WedMar2709:56:44CST2019 
mZxid=0x6a0000000a
mtime=WedMar2709:56:44CST2019
pZxid=0x6a0000000e 
cversion=2
dataVersion=0
=0x0aclVersion=0 
ephemeralOwner 
dataLength=0
numChildren=2
```

属性说明：

```
cZxid：数据节点创建时的事务ID
ctime：数据节点创建时的时间
mZxid：数据节点最后一次更新时的事务ID mtime：数据节点最后一次更新时的时间
pZxid：数据节点的子节点最后一次被修改时的事务ID
cversion：子节点的更改次数
dataVersion：节点数据的更改次数
aclVersion：节点的ACL的更改次数
ephemeralOwner：如果节点是临时节点，则表示创建该节点的会话的SessionID；如果节点是持久节点，则该属性值为0
dataLength：数据内容的长度
numChildren：数据节点当前的子节点个数
```

zookeeper中的节点有两种，分别为临时节点和永久节点。节点的类型在创建时即被确定，并且不能改变。

临时节点：该节点的生命周期依赖于创建它们的会话。一旦会话(Session)结束，临时节点将被自动删除，当然可以也可以手动删除。虽然每个临时的Znode都会绑定到一个客户端会话，但他们对所有的客户端还是可见的。另外，ZooKeeper的临时节不允许有子节点

持久化节点：该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，他们才能被删除



## 3.zookeeper**常用**Shell命令

所有命令

```
help
ZooKeeper -server host:port cmd args
        stat path [watch]
        set path data [version]
        ls path [watch]
        delquota [-n|-b] path
        ls2 path [watch]
        setAcl path acl
        setquota -n|-b val path
        history 
        redo cmdno
        printwatches on|off
        delete path [version]
        sync path
        listquota path
        rmr path
        get path [watch]
        create [-s] [-e] path data acl
        addauth scheme auth
        quit 
        getAcl path
        close 
        connect host:port

```



### 3.0 启动

```

//启动：zkServer.sh start
//停止：zkServer.sh stop
//查看状态：zkServer.sh status
./zkCli.sh  -server localhost:2181
```

客户端   对节点的操作

get/ls 获取

set 修改

create 创建

delete 删除



### 3.1新增节点

```shell
create [-s][-e] path data#其中-s为有序节点，-e临时节
```

创建持久化节点并写入数据：

```
[zk: localhost:2181(CONNECTED) 8] create /app1 "123456"
Created /app1
[zk: localhost:2181(CONNECTED) 9] get /                

cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x4
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 2
[zk: localhost:2181(CONNECTED) 10] ls / 
[zookeeper, app1]

```

创建持久化有序节点，此时创建的节点名为指定节点名+自增序号

```
[zk:localhost:2181(CONNECTED)2]create -s /a "aaa" Created /a0000000000
[zk:localhost:2181(CONNECTED)3]create -s /b "bbb" Created /b0000000001
[zk:localhost:2181(CONNECTED)4]create -s /c "ccc" Created /c0000000002
```

创建临时节点，临时节点会在会话断开后被删除，也就是，客户端开着就有，关了再打开就没了。

```
zk:localhost:2181(CONNECTED)5]create -e /tmp "tmp"
```

### 3.**2**更新节点

更新节点的命令是set，可以直接进行修改，如下：

```
[zk:localhost:2181(CONNECTED)3]set /hadoop "345" cZxid=0x4
```

​	也可以基于版本号进行更改，此时类似于乐观锁机制，当你传入的数据版本号(dataVersion)和当前节点的数据版本号不符合时，zookeeper会拒绝本次修改：

```
[zk:localhost:2181(CONNECTED)10]set /hadoop "3456" 1 versionNoisnotvalid:/hadoop
```

### 3.3 删除节点

删除节点的语法如下：

```
delete path [version]
```

和更新节点数据一样，也可以传入版本号，当你传入的数据版本号(dataVersion)和当前节点的数据版本号不符合时，zookeeper不会执行删除操作。

```
[zk:localhost:2181(CONNECTED)36]delete /hadoop 0
```

### 3.4 查看节点

```
get path
```

```
zk:localhost:2181(CONNECTED)1]get /hadoop
```

节点各个属性如下表。其中一个重要的概念是Zxid(ZooKeeperTransaction Id)，ZooKeeper节点的每一次更改都具有唯一的Zxid，如果Zxid1小于Zxid2，则Zxid1的更改发生在Zxid2更改之前。

![image-20240520235751862](https://raw.githubusercontent.com/PeipengWang/picture/master/zk/image-20240520235751862.png)

```
get /path watch
```

册的监听器能够在节点内容发生改变的时候，向客

### 3.5 查看当前节点下的子节点

```
ls  路径
[zk: localhost:2181(CONNECTED) 5] ls /          
[zookeeper]
[zk: localhost:2181(CONNECTED) 6] ls /zookeeper
[aa, quota]

```



### 3.6 监听器**ls\ls2path[watch]**

```
[zk:localhost:2181(CONNECTED)9]ls /hadoop watch
[]
[zk:localhost:2181(CONNECTED)10]create /hadoop/yarn "aaa"
```

注册的监听器能够监听该节点下所有子节点的增加和删除操作。

## 4、Java API

引入依赖

```html
<dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>zookeeper</artifactId>
                    <groupId>org.apache.zookeeper</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>log4j</artifactId>
                    <groupId>log4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>slf4j-api</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
            <version>0.9</version>
        </dependency>
        <dependency>
            <artifactId>zookeeper</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>log4j</artifactId>
                    <groupId>log4j</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
            <groupId>org.apache.zookeeper</groupId>
            <version>3.4.10</version>
        </dependency>
```

`zonde `是 `zookeeper `集合的核心组件，` zookeeper API` 提供了一小组使用 `zookeeper `集群来操作`znode `的所有细节

客户端应该遵循以下步骤，与`zookeeper`服务器进行清晰和干净的交互

- 连接到`zookeeper`服务器。`zookeeper`服务器为客户端分配会话`ID`
- 定期向服务器发送心跳。否则，`zookeeper `服务器将过期会话`ID`，客户端需要重新连接
- 只要会话`Id`处于活动状态，就可以获取/设置`znode`
- 所有任务完成后，断开与`zookeeper`服务器连接，如果客户端长时间不活动，则`zookeeper`服务器将自动断开客户端

直接写代码

```
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

import java.io.IOException;
import java.util.concurrent.CountDownLatch;

public class Test {
    public static void main(String[] args) throws IOException, InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(1);

        ZooKeeper zookeeper = new ZooKeeper("159.75.251.138:2181", 5000, (WatchedEvent x) -> {
            if (x.getState() == Watcher.Event.KeeperState.SyncConnected) {
                System.out.println("连接成功");
                countDownLatch.countDown();
            }
        });
        countDownLatch.await();
        System.out.println(zookeeper.getSessionId());
        zookeeper.close();
    }
}

```

```

连接成功
112479687956365317

Process finished with exit code 0

```

或者

![image-20240521235444155](https://raw.githubusercontent.com/PeipengWang/picture/master/zk/image-20240521235444155.png)
