# [windows环境下elasticsearch安装教程(超详细)](https://www.cnblogs.com/hualess/p/11540477.html)

# 一、安装jdk

ElasticSearch是基于lucence开发的，也就是运行需要java jdk支持。所以要先安装JAVA环境。

由于ElasticSearch 5.x 往后依赖于JDK 1.8的，所以现在我们下载JDK 1.8或者更高版本。
下载JDK1.8,下载完成后安装。

# 二、安装ElasticSearch

1.ElasticSearch下载地址：

https://www.elastic.co/downloads/elasticsearch

2.下载安装包后解压

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/es/1676314-20190918111243712-111248218.png)

 

 

 

3.进入bin目录下,双击执行elasticsearch.bat

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/es/1676314-20190918111219472-1639773577.png)

 

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/es/1676314-20190918111601780-92205386.png)

 

 

 4.看到started说明启动成功,打开浏览器测试一下,如下图

http://localhost:9200

![img](https://raw.githubusercontent.com/PeipengWang/picture/master/es/1676314-20190918111749049-1111981258.png)

#  三、安装ElasticSearch

 修改配置文件`kibana.yml`，如下：

![img](https://ask.qcloudimg.com/http-save/yehe-7276506/8c101063277508039d4c837c68ac1894.png)



```javascript
server.port:5601
server.host:""
elasticsearch.hosts:[""]
```

3.运行bin目录下的`kibana.bat`批处理文件：

![img](https://ask.qcloudimg.com/http-save/yehe-7276506/35e9e8c0a46ab08f129a91f8d5d738a9.png)

4.访问上述配置步骤中的：`ip:5601`：

![img](https://ask.qcloudimg.com/http-save/yehe-7276506/d34c5f2fe5343efd4fb956016674778b.png)

![img](https://ask.qcloudimg.com/http-save/yehe-7276506/70a75f0e1f1a4382ef9d4d9c591704f2.png)

5.实例：添加Apache logs

![img](https://ask.qcloudimg.com/http-save/yehe-7276506/52ffeeaf9aae507b931821888ed0023d.png)

# Windows安装启动logstash

(1)下载logstash，下载链接：

Download Logstash Free | Get Started Now | Elastic

（2）下载完成后，解压。自己写一个Windows环境下的配置文件，名字可自己定义，比如叫：my_logstash.conf，内容：

input {
    stdin{
    }
} 

output {
    stdout{
    }
}
注意my_logstash.conf的存放位置，这个配置文件需要放在“home”下，和logstash解压后形成的bin，config，jdk等目录文件夹同层级，否则后面启动会报找不到这个配置文件。

（3）启动logstash，启动命令：

logstash -f my_logstash.conf
其中logstash是位于bin目录下的Windows执行文件logstash.bat。

（4）如果启动成功，那么就可以在浏览器打开地址 http://localhost:9600/

返回当前logstash的基础信息。

注意：logstash要和es同一个版本号。

logstash基本功能：



Logstash包含3个主要部分： 输入(inputs)，过滤器(filters)和输出(outputs)。

