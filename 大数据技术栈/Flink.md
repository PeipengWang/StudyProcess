## Flink安装-Linux版本

下载地址：https://www.apache.org/dyn/closer.lua/flink/flink-1.9.0/flink-1.9.0-bin-scala_2.12.tgz

## 二.运行Flink

## 2.1 Java安装

运行 Flink 需要安装 Java 7.x 或更高的版本，操作系统需要 Win 7 或更高版本。

## 2.2 修改配置

修改conf/config.yaml

```
rest:
  # 外部访问的端口
  address: 152.136.246.11  
  # The address that the REST & web server binds to
  # By default, this is localhost, which prevents the REST & web server from
  # being able to communicate outside of the machine/container it is running on.
  #
  # To enable this, set the bind address to one that has access to outside-facing
  # network interface, such as 0.0.0.0.  内部要监听的地址，或者填写内网地址
  bind-address: 0.0.0.0
  # # The port to which the REST client connects to. If rest.bind-port has
  # # not been specified, then the server will bind to this port as well. 内部访问的端口
  port: 8081
  # # Port range for the REST and web server to bind to.  暴露在外的端口，提供web访问
  bind-port: 18081

```

## 2.3 运行Flink

```
./bin/start-cluster.sh
Starting cluster.
Starting standalonesession daemon on host.
Starting taskexecutor daemon on host.运行Flink的命令非常简单，只需要进入到解压目录的bin目录下，运行[start-cluster.bat](https://zhida.zhihu.com/search?content_id=187628581&content_type=Article&match_order=1&q=start-cluster.bat&zhida_source=entity)即可
```

## 三.访问 Flink UI

Flink有个UI界面，可以用于监控Flilnk的job运行状态，上一步已经给出了具体链接 http://152.136.246.11:18081/