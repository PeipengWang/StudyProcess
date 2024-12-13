# ELK安装

## Elasticseaerch安装配置

```
创建用户
useradd elk
创建所属组
chown elk:elk  elasticsearch-7.13.2-linux-x86_64.tar.gz
```

解压缩

```
tar -zxvf elasticsearch-7.13.2-linux-x86_64.tar.gz
```


切换用户

```
su elk
```

进入目录

```
cd /opt/elasticsearch-7.13.2/bin
```

修改配置文件（jdk版本与elk自带jdk版本不对应问题）

```
vim ./elasticsearch
```

```
export JAVA_HOME=/opt/elasticsearch-7.13.2/jdk
export PATH=$JAVA_HOME/bin:$PATH

if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/opt/elasticsearch-7.13.2/jdk/bin/java"
else
        JAVA=`which java`
fi
```

修改/opt/elasticsearch-7.13.2/config/elasticsearch.yml
添加如下配置：

```
network.host: 0.0.0.0
cluster.name: my-application

node.name: node-1
cluster.initial_master_nodes: ["node-1"]
```

```
xpack.license.self_generated.type: basic
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

### 问题1：

error: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

解决办法:
vi /etc/security/limits.conf
在最后几行添加

*               soft    nofile          65536
*               hard    nofile          65536

vi /etc/sysctl.conf
添加

```
vm.max_map_count=655360
```

保存后执行
sysctl -p

后台启动
 ./elasticsearch -d

elasticsearch修改密码(所有用户相同密码即可，建议：123456)：
cd /opt/elasticsearch-7.13.2/bin/
./elasticsearch-setup-passwords interactive

## 安装kibana 

解压缩

```
tar -zxvf kibana-7.13.2-linux-x86_64.tar.gz
```

修改配置文件
vi kibana-7.13.2-linux-x86_64/config/kibana.yml

```
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://172.24.1.6:9200"]
kibana.index: ".kibana"
```

后台启动kibana

```
nohup ./kibana &
```

