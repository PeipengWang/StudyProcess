

@[TOC](Docker介绍)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、Docker作用与的一些基本概念






# 二、Docker环境准备
## 1.安装Linux
可以购买阿里云或者腾讯云主机
## 2.Docker安装步骤

```shell
uname -r
```
docker要求CentOS系统的内核高于3.10
如果低于则用以下命令升级

```shell
yum update
```
安装Docker

```shell
yum install docker
```
卸载docker

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```

查看当前docker的版本号
```shell
docker -v
```
设置开机启动docker
```shell
systemctl enable docker
```
启动docker

```shell
systemctl start docker
```
重启docker

```
systemctl restart docker  # 重启docker服务
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112119523161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70#pic_center)

<font color=#999AAA >常见错误：</font>
docker-ce conflicts with 2:docker-1.13.1-203.git0be3e21.el7.centos.x86_64
已经安装过了或者卸载不彻底
查看需要卸载的文件
```shell
yum list installed | grep docker
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121195418820.png#pic_center)
卸载
```shell
yum remove -y containerd.io.x86_64 docker-ce.x86_64 docker-ce-cli.x86_64

```
删除文件
```shell
rm -rf /var/lib/docker
```
# 三. 常用操作
关闭防火墙

```
systemctl stop firewalld
```
禁止开机启动防火墙
```
systemctl disable firewalld
```

搜索
```shell
docker search 例子
例如：docker search mysql 
```
拉取
```shell
docker pull 镜像名：镜像版本
例如：docker pull mysql：5.7
```
查看本地镜像列表
```shell
docker images
```
显示内容如下格式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201121200933355.png#pic_center)
删除镜像
```shell
docker rmi IMAGE-ID 
以上述的ID作为删除对象的标志
```

# 四. 容器操作
根据镜像启动容器：
```shell
docker run --name container-name -d image-name
--name：自定义容器名
-d：后台运行
image-name：指定镜像模板
eg:docker run --mytomcat -d tomcat:latest
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201122204331578.png#pic_center)

查看运行中的镜像：
```shell
docker ps
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201122204423283.png#pic_center)
查看所有的镜像

```c
docker ps -a
```

停止程序

```shell
docker stop ID
ID是docker ps显示出来的ID
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201122204603917.png#pic_center)
启动容器
```shell
docker start 容器id
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201122204938424.png#pic_center)
删除一个容器
```shell
docker rm 容器id
```
端口映射

```shell
docker run -d -p 虚拟机端口：docker中的端口 名称
-d:表示后台运行
-p:主机的端口映射到容器的一个端口
eg：docker run -d -p 8888:8800 tomcat
这个地方需要关闭防火墙
service firewalld status：查看防火墙状态
service firewalld stop：关闭防火墙
```
查看docker的日志

```
docker logs container-name
```
进入docker

```
docker exec -it mn bash
```
docker exec ：进入容器内部，执行一个命令
-it : 给当前进入的容器创建一个标准输入、输出终端，允许我们与容器交互
mn ：要进入的容器的名称
bash：进入容器后执行的命令，bash是一个linux终端交互命令
