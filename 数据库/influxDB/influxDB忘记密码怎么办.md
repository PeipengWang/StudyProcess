

# InfluxDB忘记密码怎么解决（linux）

前提条件：

有客户端influx指令

```
[root@VM-20-15-centos influxdb2-client-2.4.0-linux-amd64]# ls
influx  LICENSE  README.md
```

**当前以2.4版本测试**

搜索influxd.bolt文件

这个位置不固定，需要具体搜索一下

```
[root@VM-20-15-centos /]# find / -name influxd.bolt
/root/.influxdbv2/influxd.bolt
```



在这个混合文本和二进制json文件中搜索您已知的用户名或`token`之类的字符串（注意要看一下description）。

```
{"id":"1234567898000000",
"token":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==",
"status":"active",
"description":"admins's Token",
```

借助管理员特权令牌，您可以使用Influx命令行界面命令`influx user password`更新口令。例如：

```
./influx user password -n admin -t xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==
```

