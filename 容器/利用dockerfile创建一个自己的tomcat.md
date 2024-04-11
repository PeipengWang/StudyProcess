
## 准备
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210602170525394.png)
下载tomcat和jdk，其中将jdk解压

```shell
tar -zxvf jdk-8u221-linux-x64.tar.gz
```
创建一个dockerfile文件

```shell
vim dockerfile
```
注意，修改可以使用vi dockerfile
## 1、编写dockerfile

```shell
FROM centos
MAINTAINER wpp<243588992@qq.com>

# 把宿主机当前目录下的 jdk1.8.0_221 拷贝到容器 /usr/local/ 路径下
COPY jdk1.8.0_221/ /usr/local/jdk1.8.0_221/

# 把宿主机当前目录下的 tomcat 添加到容器 /usr/local/ 路径下
ADD apache-tomcat-9.0.46.tar.gz /usr/local/

# 安装vim编辑器
RUN yum -y install vim

# 设置工作访问时候的 WORKDIR 路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH

# 配置 java 与 tomcat 环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_221
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.46
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.46
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

# 容器运行时监听的端口
EXPOSE  8080

# 启动时运行 tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.24/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.24/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.46/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.46/bin/logs/catalina.out

```

## 2.生成镜像
```shell
docker build -f dockerfile -t mytomcat:0.1
```
其中如果卡在yun -y install vim 这一步可能要考虑docker网络问题，解决方法如下所示。

```shell
firewall-cmd --zone=public --add-masquerade --permanent
firewall-cmd --reload
systemctl restart docker
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210602171609922.png)
## 3.运行容器
```shell
  docker run -d -p 9090:8080 --name mytomcat -v /wpp/tomcat/logs/:/usr/local/apache-tomcat-9.0.24/logs  mytomcat:0.1
```
/wpp/tomcat/logs/是自己设定的挂载日志的目录，其它重要文件也可以这样挂载。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210602171646309.png)

## 4.测试
访问自己服务器的9090端口可以得到下面所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210602171520555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

