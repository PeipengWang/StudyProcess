
# 理解docker0
测试docker网络的地址
```shell
ip addr
```

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/dockerbf120610c902ef653fcac183a6c027a0.png)
下载启动一个tomcat来测试

```shell
docker pull tomcat
docker run  -d -P --name tomcat01 tomcat
```
再次测试ip发现多了一个71: vethf1870a2@if70，如图所示
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker3556b2b86833d1307aadce717ea2090b.png)
同时进一步的在容器内部查看网卡，发现

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker549d29b5fcefbc6120d6eafbd47ddfa0.png)
正好与docker内的网卡想对应，由此可以看出，docker容器可以直接与tomcat进行联通，验证如下所示。其中172.17.0.2为tomcat的内部网卡。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker97aebe044bef7ea494fb412ca0db8672.png)
同时在tomcat容器中ping docker也是可以的，如下图所示，其中172.17.0.1是docker容器内的docker0网卡
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker8d90bba2f12237c406fa300ea20faafb.png)
**通过上述测试可以知道docker容器是可以与tomcat建立连接的**，那么两个tomcat是否可以进行网络连接呢？
首先需要再次启动一个tomcat容器

```shell
docker run -d -P --name tomcat02 tomcat
```
这时候测试网卡

```
ip addr
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/dockerb1292421139e9f048c5e85ba9a0fdc6d.png)
发现又多了一块。
进一步测试新开的容器tomcat的网卡发现，正好与docker的网卡相对应。由此可知：**每当新开一个容器，docker总会生成一个相对应的一对网卡连接地址**，这是一种veth-pair技术，这种技术，一端连接协议，一端彼此相连。正因为这种技术，veth-pair充当桥梁使得容器可以彼此相连。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker5070a4c21b9758fec69fd7d1120d4965.png)
测试：查看tomcat02是否能够ping通tomcat01

```shell
 docker exec -it tomcat02 ping 172.17.0.2
```

![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker599f691dcce53e663650b6b6518424ad.png)
由上可知，没生成一个容器会在docker0中生成一个地址分配给容器，每个容器能够与docker0相连，由此每个容器之间可以通过docker0进行间接通信。docker0也可以称作是容器之间相连的网桥。

# link
我们也可以直接利用名字进行容器之间的联通
首先，如果直接执行名字互联，执行如下命令

```
 docker exec -it tomcat02 ping tomcat01
```
出现`
ping: tomcat01: Name or service not known
`
这是因为无法进行，名字之间的联通，因此可以利用link命令解决这个问题
新建一个tomcat03来link tomcat01
```
 docker run -d -P --name tomcat03 --link tomcat01 tomcat
```
执行

```
docker exec -it tomcat03 ping tomcat01
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/dockere3efa1aecd35195de32e080d1d96ff8f.png)
发现tomcat03可以ping通tomcat01

究其本质
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker8cd27faae24a767b2464de48e189be00.png)
观察tomcat03的hosts文件可以发现tomcat01名字本质上是ip地址，因此tomcat03可以直接通过名字连接tomcat01的。相反tomcat01是无法直接通过名字连接tomcat03的，原因如下图所示。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker6b962e65a487e6627eca88db0e18ec03.png)
可以发现tomcat01中没有关于tomcat03的名字记录的。

# 自定义网络
为了解决上述无法进行双向利用名字连接的问题，将会利用自定义网络的方式来进行连接。
查看docker所有网络

```
docker network ls
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/dockereee9c72de581d50a39d5aac67da2efbe.png)
bridge：桥接模式
host：和宿主机共享网络
none: 不配置网络

自定义配置网络
```
docker network create --driver bridge --subnet 192.168.0.0/24 --gateway 192.168.0.1 mynet
--driver bridge：驱动模式，桥接
--subnet 192.168.0.0/24：子网地址
--gateway 192.168.0.1：路由器地址
 mynet：自定义网络名称
```
查看自己建立的网络
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/dockere927f68f55165049dfcaba8e0aa773ef.png)

```
[root@VM-0-4-centos ~]# docker network inspect b6e6618b56af
[
    {
        "Name": "mynet",
        "Id": "b6e6618b56afdb8e836e7ce2c8bb1f70bca58cae1c879d38ac4197b756203646",
        "Created": "2021-06-03T15:59:04.502780878+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/24",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

```
创建容器来测试

```
docker run -d -P --name tomcat01 --net mynet tomcat
docker run -d -P --name tomcat02 --net mynet tomcat
[root@VM-0-4-centos ~]# docker exec -it tomcat01 ping tomcat02
PING tomcat02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.069 ms
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.065 ms
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.080 ms
[root@VM-0-4-centos ~]# docker exec -it tomcat02 ping tomcat01
PING tomcat01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.065 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.067 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=4 ttl=64 time=0.076 ms

```
发现tomcat01与tomcat02可以互相通过名字来ping通，进一步观察tomcat01的网卡
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker8d54c65f1480d48ef61bd3e6d48d2217.png)
如果建立一个默认的tomcat03

```

[root@VM-0-4-centos ~]# docker run -d -P --name tomcat03 tomcat
6172b2bcbdb39e8edea3134688e2c9c37e110e9fae01f4dbf8bbc9e3d5588009
[root@VM-0-4-centos ~]# docker exec -it tomcat03 ping tomcat01
ping: tomcat01: Name or service not known
```
如上所示，tomcat03是无法打通tomcat01的，其原因如图所示，其中tomcat03与tomcat01不在一个网络内，因此要想实现tomcat03 ping通mynet子网内的容器，只能将tomcat03连接到mynet下。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker534263e4a5cbfc7e330213057f1251ee.png)

```
[root@VM-0-4-centos ~]# docker network connect mynet tomcat03
[root@VM-0-4-centos ~]# docker exec -it tomcat03 ping tomcat01
PING tomcat01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.065 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.089 ms
64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.079 ms

```
通过上述可以发现可以ping通，这时候需要注意的是tomcat03与子网mynet所属的网络联通了。