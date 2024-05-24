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

### 4.1 引入依赖

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

### 4.2 定义客户端

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

### 4.3 创建节点

##### 新增节点

```
// 同步
create(String path, byte[] data, List<ACL> acl, CreateMode createMode)
// 异步
create(String path, byte[] data, List<ACL> acl, CreateMode createMode,
      AsynCallback.StringCallback callBack, Object ctx)
```



 | 参数         | 解释                                                         |
  | ------------ | ------------------------------------------------------------ |
  | `path`       | `znode `路径                                                 |
  | `data`       | 数据                                                         |
  | `acl`        | 要创建的节点的访问控制列表。`zookeeper API `提供了一个静态接口 `ZooDefs.Ids` 来获取一些基本的`acl`列表。例如，`ZooDefs.Ids.OPEN_ACL_UNSAFE`返回打开`znode`的`acl`列表 |
  | `createMode` | 节点的类型，这是一个枚举                                     |
  | `callBack`   | 异步回调接口                                                 |
  | `ctx`        | 传递上下文参数                                               |

示例：

 ```java
  // 枚举的方式
      public static void createTest1() throws Exception{
          String str = "node";
          String s = zookeeper.create("/node", str.getBytes(),
                  ZooDefs.Ids.READ_ACL_UNSAFE, CreateMode.PERSISTENT);
          System.out.println(s);
      }
 ```

 ```java
  // 自定义的方式
      public static void createTest2() throws Exception{
          ArrayList<ACL> acls = new ArrayList<>();
          Id id = new Id("ip","192.168.133.133");
          acls.add(new ACL(ZooDefs.Perms.ALL,id));
          zookeeper.create("/create/node4","node4".getBytes(),acls,CreateMode.PERSISTENT);
      }
 ```

 ```java
  // auth
      public static void createTest3() throws  Exception{
          zookeeper.addAuthInfo("digest","itcast:12345".getBytes());
          zookeeper.create("/node5","node5".getBytes(),
                  ZooDefs.Ids.CREATOR_ALL_ACL,CreateMode.PERSISTENT);
      }
  // 自定义的方式
      public static void createTest3() throws  Exception{
  //        zookeeper.addAuthInfo("digest","itcast:12345".getBytes());
  //        zookeeper.create("/node5","node5".getBytes(),
  //                ZooDefs.Ids.CREATOR_ALL_ACL,CreateMode.PERSISTENT);
          zookeeper.addAuthInfo("digest","itcast:12345".getBytes());
          List<ACL> acls = new ArrayList<>();
          Id id = new Id("auth","itcast");
          acls.add(new ACL(ZooDefs.Perms.READ,id));
          zookeeper.create("/create/node6","node6".getBytes(),
                  acls,CreateMode.PERSISTENT);
      }
 ```

 ```java
  // digest 
  public static void createTest3() throws  Exception{
      List<ACL> acls = new ArrayList<>();
      Id id = new Id("digest","itcast:qUFSHxJjItUW/93UHFXFVGlvryY=");
      acls.add(new ACL(ZooDefs.Perms.READ,id));
      zookeeper.create("/create/node7","node7".getBytes(), 	
                       acls,CreateMode.PERSISTENT);
  }
 ```

```java
  // 异步
      public static void createTest4() throws  Exception{
          zookeeper.create("/node12", "node12".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT, new AsyncCallback.StringCallback(){
              /**
               * @param rc 状态，0 则为成功，以下的所有示例都是如此
               * @param path 路径
               * @param ctx 上下文参数
               * @param name 路径
               */
              public void processResult(int rc, String path, Object ctx, String name){
                  System.out.println(rc + " " + path + " " + name +  " " + ctx);
              }
          }, "I am context");
          TimeUnit.SECONDS.sleep(1);
          System.out.println("结束");
      }
```

  

### 4.4 查看节点

#### 同步、异步查看节点

 ```java
  // 同步
  getData(String path, boolean watch, Stat stat)
  getData(String path, Watcher watcher, Stat stat)
  // 异步
  getData(String path, boolean watch, DataCallback callBack, Object ctx)
  getData(String path, Watcher watcher, DataCallback callBack, Object ctx)
 ```

 

 | 参数       | 解释                             |
  | ---------- | -------------------------------- |
  | `path`     | 节点路径                         |
  | `boolean`  | 是否使用连接对象中注册的监听器   |
  | `stat`     | 元数据                           |
  | `callBack` | 异步回调接口，可以获得状态和数据 |
  | `ctx`      | 传递上下文参数                   |

 ```java
      public static void getData1() throws Exception {
          Stat stat = new Stat();
          byte[] data = zookeeper.getData("/hadoop", false, stat);
          System.out.println(new String(data));
          // 判空
          System.out.println(stat.getCtime());
      }
  
      public static void getData2() throws Exception {
          zookeeper.getData("/hadoop", false, new AsyncCallback.DataCallback() {
              @Override
              public void processResult(int rc, String path, Object ctx, byte[] bytes, Stat stat) {
                  // 判空
                  System.out.println(rc + " " + path
                                     + " " + ctx + " " + new String(bytes) + " " + 
                                     stat.getCzxid());
              }
          }, "I am context");
          TimeUnit.SECONDS.sleep(3);
      }
 ```

  

#### 查看子节点

同步、异步

 ```java
  // 同步
  getChildren(String path, boolean watch)
  getChildren(String path, Watcher watcher)
  getChildren(String path, boolean watch, Stat stat)    
  getChildren(String path, Watcher watcher, Stat stat)
  // 异步
  getChildren(String path, boolean watch, ChildrenCallback callBack, Object ctx)    
  getChildren(String path, Watcher watcher, ChildrenCallback callBack, Object ctx)
  getChildren(String path, Watcher watcher, Children2Callback callBack, Object ctx)    
  getChildren(String path, boolean watch, Children2Callback callBack, Object ctx)
 ```



 | 参数       | 解释                       |
  | ---------- | -------------------------- |
  | `path`     | 节点路径                   |
  | `boolean`  |                            |
  | `callBack` | 异步回调，可以获取节点列表 |
  | `ctx`      | 传递上下文参数             |

 ```java
      public static void getChildren_1() throws Exception{
          List<String> hadoop = zookeeper.getChildren("/hadoop", false);
          hadoop.forEach(System.out::println);
      }
  
      public static void getChildren_2() throws Exception {
          zookeeper.getChildren("/hadoop", false, new AsyncCallback.ChildrenCallback() {
              @Override
              public void processResult(int rc, String path, Object ctx, List<String> list) {
                  list.forEach(System.out::println);
                  System.out.println(rc + " " + path + " " + ctx);
              }
          }, "I am children");
          TimeUnit.SECONDS.sleep(3);
      }
 ```



#### 检查节点是否存在

同步、异步

 ```java
  // 同步
  exists(String path, boolean watch)
  exists(String path, Watcher watcher)
  // 异步
  exists(String path, boolean watch, StatCallback cb, Object ctx)
  exists(String path, Watcher watcher, StatCallback cb, Object ctx)
 ```



 | 参数       | 解释                       |
  | ---------- | -------------------------- |
  | `path`     | 节点路径                   |
  | `boolean`  |                            |
  | `callBack` | 异步回调，可以获取节点列表 |
  | `ctx`      | 传递上下文参数             |

 ```java
  public static void exists1() throws Exception{
      Stat exists = zookeeper.exists("/hadoopx", false);
      // 判空
      System.out.println(exists.getVersion() + "成功");
  }
  public static void exists2() throws Exception{
      zookeeper.exists("/hadoopx", false, new AsyncCallback.StatCallback() {
          @Override
          public void processResult(int rc, String path, Object ctx, Stat stat) {
              // 判空
              System.out.println(rc + " " + path + " " + ctx +" " + stat.getVersion());
          }
      }, "I am children");
      TimeUnit.SECONDS.sleep(1);
  }
 ```

### 4.5 修改节点

同样也有两种修改方式(`异步和同步`)

 ```java
  // 同步
  setData(String path, byte[] data, int version)
  // 异步
  setData(String path, byte[] data, int version, StatCallback callBack, Object ctx)
 ```



 | 参数       | 解释                                                         |
  | ---------- | ------------------------------------------------------------ |
  | `path`     | 节点路径                                                     |
  | `data`     | 数据                                                         |
  | `version`  | 数据的版本号， -`1`代表不使用版本号，乐观锁机制              |
  | `callBack` | 异步回调 `AsyncCallback.StatCallback`，和之前的回调方法参数不同，这个可以获取节点状态 |
  | `ctx`      | 传递上下文参数                                               |

 ```java
      public static void setData1() throws Exception{
      	// arg1:节点的路径
          // arg2:修改的数据
          // arg3:数据的版本号 -1 代表版本号不参与更新
          Stat stat = zookeeper.setData("/hadoop","hadoop-1".getBytes(),-1);
      }
 ```

 ```java
      public static void setData2() throws Exception{
          zookeeper.setData("/hadoop", "hadoop-1".getBytes(), 3 ,new AsyncCallback.StatCallback(){
              @Override
              public void processResult(int rc, String path, Object ctx, Stat stat) {
                  // 讲道理，要判空
                  System.out.println(rc + " " + path + " " + stat.getVersion() +  " " + ctx);
              }
          }, "I am context");
      }
 ```

### 4.5 删除节点

异步、同步

 ```java
  // 同步
  delete(String path, int version)
  // 异步
  delete(String path, int version, AsyncCallback.VoidCallback callBack, Object ctx)
 ```

 

 | 参数       | 解释                                            |
  | ---------- | ----------------------------------------------- |
  | `path`     | 节点路径                                        |
  | `version`  | 版本                                            |
  | `callBack` | 数据的版本号， -`1`代表不使用版本号，乐观锁机制 |
  | `ctx`      | 传递上下文参数                                  |

 ```java
      public static void deleteData1() throws Exception {
          zookeeper.delete("/hadoop", 1);
      }
  
      public static void deleteData2() throws Exception {
          zookeeper.delete("/hadoop", 1, new AsyncCallback.VoidCallback() {
              @Override
              public void processResult(int rc, String path, Object ctx) {
                  System.out.println(rc + " " + path + " " + ctx);
              }
          }, "I am context");
          TimeUnit.SECONDS.sleep(1);
      }
 ```

  还有几种：

删除包含子节点、必须删除成功



## 5、事件监听机制

**watcher概念**

<https://zookeeper.apache.org/doc/r3.4.14/zookeeperProgrammers.html#sc_WatchRememberThese>

- `zookeeper`提供了数据的`发布/订阅`功能，多个订阅者可同时监听某一特定主题对象，当该主题对象的自身状态发生变化时例如节点内容改变、节点下的子节点列表改变等，会实时、主动通知所有订阅者
- `zookeeper`采用了 `Watcher`机制实现数据的发布订阅功能。该机制在被订阅对象发生变化时会异步通知客户端，因此客户端不必在 `Watcher`注册后轮询阻塞，从而减轻了客户端压力
- `watcher`机制事件上与观察者模式类似，也可看作是一种观察者模式在分布式场景下的实现方式

Zookeeper提供了三种Watcher：

- NodeCache：只是监听一个特定的节点
- PathChildrenCache：监听一个ZNode的子节点
- TreeCache：可以监控整个树上的所有节点，类似于PathChildrenCache和NodeCache的组合

#### watcher架构

`watcher`实现由三个部分组成

- `zookeeper`服务端
- `zookeeper`客户端
- 客户端的`ZKWatchManager对象`

客户端**首先将 `Watcher`注册到服务端**，同时将 `Watcher`对象**保存到客户端的`watch`管理器中**。当`Zookeeper`服务端监听的数据状态发生变化时，服务端会**主动通知客户端**，接着客户端的 `Watch`管理器会**触发相关 `Watcher`**来回调相应处理逻辑，从而完成整体的数据 `发布/订阅`流程



#### watcher特性



| 特性           | 说明                                                         |
  | -------------- | ------------------------------------------------------------ |
  | 一次性         | `watcher`是**一次性**的，一旦被触发就会移除，再次使用时需要重新注册 |
  | 客户端顺序回调 | `watcher`回调是**顺序串行**执行的，只有回调后客户端才能看到最新的数据状态。一个`watcher`回调逻辑不应该太多，以免影响别的`watcher`执行 |
  | 轻量级         | `WatchEvent`是最小的通信单位，结构上只包含**通知状态、事件类型和节点路径**，并不会告诉数据节点变化前后的具体内容 |
  | 时效性         | `watcher`只有在当前`session`彻底失效时才会无效，若在`session`有效期内快速重连成功，则`watcher`依然存在，仍可接收到通知； |

**watcher接口设计**

`Watcher`是一个接口，任何实现了`Watcher`接口的类就算一个新的`Watcher`。`Watcher`内部包含了两个枚举类：`KeeperState`、`EventType`



##### Watcher通知状态(KeeperState)

`KeeperState`是客户端与服务端**连接状态**发生变化时对应的通知类型。路径为`org.apache.zookeeper.Watcher.EventKeeperState`，是一个枚举类，其枚举属性如下：

 

| 枚举属性        | 说明                     |
  | --------------- | ------------------------ |
  | `SyncConnected` | 客户端与服务器正常连接时 |
  | `Disconnected`  | 客户端与服务器断开连接时 |
  | `Expired`       | 会话`session`失效时      |
  | `AuthFailed`    | 身份认证失败时           |

  

##### Watcher事件类型(EventType)

`EventType`是**数据节点`znode`发生变化**时对应的通知类型。**`EventType`变化时`KeeperState`永远处于`SyncConnected`通知状态下**；当`keeperState`发生变化时，`EventType`永远为`None`。其路径为`org.apache.zookeeper.Watcher.Event.EventType`，是一个枚举类，枚举属性如下：

 

 | 枚举属性              | 说明                                                        |
  | --------------------- | ----------------------------------------------------------- |
  | `None`                | 无                                                          |
  | `NodeCreated`         | `Watcher`监听的数据节点被创建时                             |
  | `NodeDeleted`         | `Watcher`监听的数据节点被删除时                             |
  | `NodeDataChanged`     | `Watcher`监听的数据节点内容发生更改时(无论数据是否真的变化) |
  | `NodeChildrenChanged` | `Watcher`监听的数据节点的子节点列表发生变更时               |

 注意：客户端接收到的相关事件通知中只包含状态以及类型等信息，不包含节点变化前后的具体内容，变化前的数据需业务自身存储，变化后的数据需要调用`get`等方法重新获取

##### 捕获相应的事件

上面讲到`zookeeper`客户端连接的状态和`zookeeper`对`znode`节点监听的事件类型，下面我们来讲解如何建立`zookeeper`的***`watcher`监听***。在`zookeeper`中采用`zk.getChildren(path,watch)、zk.exists(path,watch)、zk.getData(path,watcher,stat)`这样的方式来为某个`znode`注册监听 。

下表以`node-x`节点为例，说明调用的注册方法和可用监听事件间的关系：

| 注册方式                            | created | childrenChanged | Changed | Deleted |
| ----------------------------------- | ------- | --------------- | ------- | ------- |
| `zk.exists("/node-x",watcher)`      | 可监控  |                 | 可监控  | 可监控  |
| `zk.getData("/node-x",watcher)`     |         |                 | 可监控  | 可监控  |
| `zk.getChildren("/node-x",watcher)` |         | 可监控          |         | 可监控  |

**注册watcher的方法**

##### 客户端与服务器端的连接状态

- `KeeperState `：通知状态

- `SyncConnected`：客户端与服务器正常连接时

- `Disconnected`：客户端与服务器断开连接时

- `Expired`：会话`session`失效时

- `AuthFailed`：身份认证失败时

- 事件类型为：`None`

  - 案例

  ```java
    public class ZkConnectionWatcher implements Watcher {
        @Override
        public void process(WatchedEvent watchedEvent) {
            Event.KeeperState state = watchedEvent.getState();
            if(state == Event.KeeperState.SyncConnected){
                // 正常
                System.out.println("正常连接");
            }else if (state == Event.KeeperState.Disconnected){
                // 可以用Windows断开虚拟机网卡的方式模拟
                // 当会话断开会出现，断开连接不代表不能重连，在会话超时时间内重连可以恢复正常
                System.out.println("断开连接");
            }else if (state == Event.KeeperState.Expired){
                // 没有在会话超时时间内重新连接，而是当会话超时被移除的时候重连会走进这里
                System.out.println("连接过期");
            }else if (state == Event.KeeperState.AuthFailed){
                // 在操作的时候权限不够会出现
                System.out.println("授权失败");
            }
            countDownLatch.countDown();
        }
        private static final String IP = "192.168.133.133:2181"
    ;
        private static CountDownLatch countDownLatch = new CountDownLatch(1);
    
        public static void main(String[] args) throws Exception {
            // 5000为会话超时时间
            ZooKeeper zooKeeper = new ZooKeeper(IP, 5000, new ZkConnectionWatcher());
            countDownLatch.await();
            // 模拟授权失败
            zooKeeper.addAuthInfo("digest1","itcast1:123451".getBytes());
            byte[] data = zooKeeper.getData("/hadoop", false, null);
            System.out.println(new String(data));
            TimeUnit.SECONDS.sleep(50);
        }
    }
  ```



##### watcher检查节点

**exists**

- `exists(String path, boolean b)`

- `exists(String path, Watcher w)`

- `NodeCreated`：**节点**创建

- `NodeDeleted`：**节点**删除

- `NodeDataChanged`：**节点**内容

   案例

   ```java
    public class EventTypeTest {
        private static final String IP = "192.168.133.133:2181";
        private static CountDownLatch countDownLatch = new CountDownLatch(1);
        private static ZooKeeper zooKeeper;
    
        // 采用zookeeper连接创建时的监听器
        public static void exists1() throws Exception{
            zooKeeper.exists("/watcher1",true);
        }
        // 自定义监听器
        public static void exists2() throws Exception{
            zooKeeper.exists("/watcher1",(WatchedEvent w) -> {
                System.out.println("自定义" + w.getType());
            });
        }
        // 演示使用多次的监听器
        public static void exists3() throws Exception{
            zooKeeper.exists("/watcher1", new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    try {
                        System.out.println("自定义的" + watchedEvent.getType());
                    } finally {
                        try {
                            zooKeeper.exists("/watcher1",this);
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
        }
        // 演示一节点注册多个监听器
        public static void exists4() throws Exception{
            zooKeeper.exists("/watcher1",(WatchedEvent w) -> {
                System.out.println("自定义1" + w.getType());
            });
            zooKeeper.exists("/watcher1", new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    try {
                        System.out.println("自定义2" + watchedEvent.getType());
                    } finally {
                        try {
                            zooKeeper.exists("/watcher1",this);
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }
            });
        }
        // 测试
        public static void main(String[] args) throws Exception {
            zooKeeper = new ZooKeeper(IP, 5000, new ZKWatcher());
            countDownLatch.await();
            exists4();
            TimeUnit.SECONDS.sleep(50);
        }
    
        static class ZKWatcher implements Watcher{
            @Override
            public void process(WatchedEvent watchedEvent) {
                countDownLatch.countDown();
                System.out.println("zk的监听器" + watchedEvent.getType());
            }
        }
    
    }
   ```

**getData**

- `getData(String path, boolean b, Stat stat)`
- `getData(String path, Watcher w, Stat stat)`
- `NodeDeleted`：**节点**删除
- `NodeDataChange`：**节点**内容发生变化

**getChildren**

- `getChildren(String path, boolean b)`
- `getChildren(String path, Watcher w)`
- `NodeChildrenChanged`：**子节点**发生变化
- `NodeDeleted`：**节点删除**

### 

- 
