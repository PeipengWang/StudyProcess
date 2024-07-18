# 下载安装



## 环境准备

提前准备一台CentOS7，我的配置为 `2c4g50g`，为了方便，我会直接关闭服务器的防火墙，执行以下代码：

```shell
systemctl stop firewalld
systemctl disable firewalld
```



```shell
wget \
https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.7-linux-x86_64.tar.gz \
https://artifacts.elastic.co/downloads/kibana/kibana-7.17.7-linux-x86_64.tar.gz \
https://artifacts.elastic.co/downloads/logstash/logstash-7.17.7-linux-x86_64.tar.gz

```



分词器下载

```
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.17.7/elasticsearch-analysis-ik-7.17.7.zip
```

安装jdk

```
yum -y install java-8
```

## 安装ElasticSearch

我准备将ElasticSearch安装在 /opt/server 文件夹下，所以我们创建这个文件夹：

```
mkdir -p /opt/server
```


然后我们解压 elasticsearch-7.17.7-linux-x86_64.tar.gz 文件到这个文件夹：

```
tar -zxvf elasticsearch-7.17.7-linux-x86_64.tar.gz -C /opt/server/
```

我们来到 /opt/server 目录可以查看到解压的

因为es要求不能以root用户运行该应用，所以我们为es创建一个用户 elk，并将 `elasticsearch-7.17.7` 目录和下面所有文件的所有权转到用户elk上：

```sh
# 创建用户
useradd elk
# 将所有权赋予给elk用户
chown elk:elk -R elasticsearch-7.17.7
```

随后我们需要改一些系统的配置文件，更详细的说明相见我的[es安装教程](https://blog.csdn.net/m0_51510236/article/details/120829135)，这里我们就直接执行以下代码配置：

```
echo "elk hard nofile 65536" >> /etc/security/limits.conf
echo "elk soft nofile 65536" >> /etc/security/limits.conf
echo "vm.max_map_count=655360" >> /etc/sysctl.conf
sysctl -p
```

修改JVM的内存大小，修改 `elasticsearch-7.14.2/config/jvm.options` 文件，修改内存大小，可根据物理机实际配置修改

```
-Xms256m
-Xmx256m
-Xmn128m
```

修改 `elasticsearch-7.14.2/config/elasticsearch.yml` 文件，修改配置（`10.0.16.13` 为我自己的腾讯云IP内网地址，公网地址一直绑定失败）：

```yml
cluster.name: "my-application"
node.name: "node-1"
network.host: 10.0.16.13
http.port: 9200
transport.port: 9300-9400

discovery.seed_hosts: ["43.143.251.77"]
cluster.initial_master_nodes: ["node-1"]

```

启动

```
nohup ./elasticsearch > run.log 2>&1 &
```

访问：

http://43.143.251.77:9200/

```
{
  "name" : "node-1",
  "cluster_name" : "my-application",
  "cluster_uuid" : "iQlPDolPSZWzOWLsnpQQ3g",
  "version" : {
    "number" : "7.17.7",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "78dcaaa8cee33438b91eca7f5c7f56a70fec9e80",
    "build_date" : "2022-10-17T15:29:54.167373105Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

elasticsearch安装成功

## 安装Kibana

我准备将Kibana安装在 `/opt/server` 文件夹下，我们已经创建了这个文件夹，所以直接解压文件(注意切换到root用户)：

```
# 解压文件到指定文件夹
tar -zxvf kibana-7.17.7-linux-x86_64.tar.gz -C /opt/server/
```

同样来到 `/opt/server` 文件夹下，将 `kibana-7.17.7-linux-x86_64` 文件夹以及下面的文件的所有权授予给 elk：

```
cd /opt/server/
chown elk:elk -R kibana-7.17.7-linux-x86_64/
```

此时我们需要修改 `config/kibana.yml` 配置文件，修改的内容为：

```yml
# kibana地址，注意修改为自己的服务器地址（需要取消注释）
server.host: "10.0.16.13"  #自己的内网
# elasticsearch地址，注意修改为自己的es服务器地址（需要取消注释）
elasticsearch.hosts: ["http://公网IP:9200"]
# 国际化地址修改为中文（需要取消注释）
i18n.locale: "zh-CN"
```

运行

```sh
 nohup ./kibana > run.log 2>&1 &
```

访问地址为：http://43.143.251.77:5601/

## 安装Logstash

我准备将Logstash安装在 `/opt/server` 文件夹下，我们已经创建了这个文件夹，所以直接解压文件(注意切换到root用户)：

```bash
# 解压文件到指定文件夹
tar -zxvf logstash-7.17.7-linux-x86_64.tar.gz -C /opt/server/
```

同样来到 `/opt/server` 文件夹下，将 `logstash-7.17.7` 文件夹以及下面的文件的所有权授予给 elk：

```bash
cd /opt/server/
chown elk:elk -R logstash-7.17.7/
```

同样我们需要修改一下 `config/logstash.yml` 配置文件，需要修改的内容：

```bash
# 节点的名称，取一个好听的名字（需要取消注释）
node.name: test-log
# pipeline 配置文件的路径，可自行修改，最好是空文件夹（需要取消注释）
path.config: /opt/server/logstash-7.17.7/config/conf/*.conf
```

同样为了避免内存不足的问题，我们需要修改一下 `config/jvm.options` 配置文件，在文件中添加这几行代码(参数可以根据自己的内存大小自行修改)：

```bash
-Xms256m
-Xmx256m
-Xmn128m
```

然后我们再到我们上面 `path.config` 后面配置的文件夹(我配置的文件夹是/opt/server/logstash-7.17.7/config/conf/)中创建一个 `test-log.conf` 文件，文件内容为：

```
input {
  tcp {
    mode => "server"
    port => 4560
  }
}
filter {}
output {
  elasticsearch {
    action => "index"
    hosts  => ["192.168.3.233:9200"]
    index  => "test-log"
  }
}
```

文件中包含了以下几个模块：

input：日志的输出来源，我们将暴露一个4560端口接收来自SpringBoot的日志
filter：日志的过滤器，暂时不配置
output：日志的输出目的地，我们将日志输出到elasticsearch中进行保存，如果有多个es可以在中括号当中填写多个，以逗号隔开，其中index配置的test-log即为存储日志用到的索引名称，可自行修改然后我们切换到elk用户去启动logstash：

```bash
cd logstash-7.17.7/
su elk
nohup ./logstash > run.log 2>&1 &
```



# 新建项目

新建一个SpringBoot项目

目录结构为

![在这里插入图片描述](https://img-blog.csdnimg.cn/d32b424217cb4991bcaa8f2e628918a7.png#pic_center)

## 引入依赖和配置

```xml
 <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.6.13</spring-boot.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>7.3</version>
        </dependency>

    </dependencies>

```

Springboot配置

```yaml
server:
  port: 8080

log:
  # logstash 地址和端口，注意修改
  logstash-host: 152.136.246.11:4560

```

logback-spring.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 日志存放路径 -->
    <property name="log.path" value="logs/test-log"/>
    <!-- 日志输出格式 -->
    <property name="log.pattern" value="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{20} - [%method,%line] - %msg%n"/>
    <!-- 读取SpringBoot配置文件获取logstash的地址和端口 -->
    <springProperty scope="context" name="logstash-host" source="log.logstash-host"/>

    <!-- 控制台输出 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>

    <!-- 系统日志输出 -->
    <appender name="file_info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/info.log</file>
        <!-- 循环政策：基于时间创建日志文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件名格式 -->
            <fileNamePattern>${log.path}/info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志最大的历史 7天 -->
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的级别 -->
            <level>INFO</level>
            <!-- 匹配时的操作：接收（记录） -->
            <onMatch>ACCEPT</onMatch>
            <!-- 不匹配时的操作：拒绝（不记录） -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="file_error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/error.log</file>
        <!-- 循环政策：基于时间创建日志文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件名格式 -->
            <fileNamePattern>${log.path}/error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志最大的历史 60天 -->
            <maxHistory>60</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的级别 -->
            <level>ERROR</level>
            <!-- 匹配时的操作：接收（记录） -->
            <onMatch>ACCEPT</onMatch>
            <!-- 不匹配时的操作：拒绝（不记录） -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 将日志文件输出到Logstash -->
    <appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <!-- 获取logstash地址作为输出的目的地 -->
        <destination>${logstash-host}</destination>
        <encoder chatset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>

    <!-- 系统模块日志级别控制  -->
    <logger name="com.greateme" level="info"/>
    <!-- Spring日志级别控制  -->
    <logger name="org.springframework" level="warn"/>

    <root level="info">
        <appender-ref ref="console"/>
    </root>

    <!--系统操作日志-->
    <root level="info">
        <appender-ref ref="file_info"/>
        <appender-ref ref="file_error"/>
        <appender-ref ref="logstash"/>
    </root>
</configuration>

```

## 测试代码

建立Controller

```java
package com.example.elk.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 * 测试日志的Controller
 * </p>
 *
 * @author XiaoHH
 * @version 1.0.0
 * @date 2023-04-26 22:59:13
 * @file TestLogController.java
 */
@RestController
public class TestLogController {

    /**
     * 获取日志输出对象
     */
    private static final Logger log = LoggerFactory.getLogger(TestLogController.class);

    /**
     * 测试输出log的访问方法
     */
    @GetMapping("/testLog")
    public String testLog() {
        log.error("测试输出一个日志");
        return "success";
    }
}

```

我们来启动项目，我们可以发现启动时就输出了八条日志：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a784be59ddd14bc9902cfaec0b48027c.png#pic_center)

我们来到kibana查询一下索引列表：

![在这里插入图片描述](https://img-blog.csdnimg.cn/8311f69eef1142d9895e18c2b1cc60e2.png#pic_center)

然后再看看里面的数据，发现的确新增了8条内容：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2e1cc3e2714741faa4b1bf5964ab0241.png#pic_center)



我们编写了一个Controller也会输出日志，我们访问试试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/5d86c354bea9475484d4ca69edb9d2b2.png#pic_center)

上面日志输出成功了，我们再来看看es里面的数据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/833802aa9eee44f39793d453c4f52db6.png#pic_center)

同时也可以看到日志的内容可以看到我们自定义输出的日志：

![在这里插入图片描述](https://img-blog.csdnimg.cn/7ae0f67774de4902b21bdc09fe678ebe.png#pic_center)

代码仓库地址：
https://gitcode.net/m0_51510236/test-log

```
GET test-log/_search
{
"sort": [
		{  
		"@timestamp": {
		"order": "desc"
		}
		}
	]
}

```

