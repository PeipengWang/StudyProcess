### 步骤0：要用到influx命令，先检查命令是否存在

不存在则需要安装influxDB管理工具，解压influxdb2-client-2.7.5-linux-arm64.tar.gz

```
yum install influxdb2-cli -y
```

### 步骤 1：定位 influxd.bolt 文件

默认路径（最常用）：

```
/var/lib/influxdb2/influxd.bolt
```

如果找不到，用搜索命令查找：

```
# 全服务器搜索
find / -name "influxd.bolt" 2>/dev/null
```

### 步骤 2：从 bolt 文件中提取管理员 Token

`influxd.bolt` 是二进制 + 文本混合文件，直接用 `grep` 搜索**管理员 Token**即可：

```
grep -a "token" /var/lib/influxdb2/influxd.bolt
```

或者直接打开这个文件看看即可

你会看到类似格式的内容（提取 token 值）

```
{"id":"1234567890000000","token":"abcdefghijklmnopqrstuvwxyz1234567890==","status":"active","description":"admin's Token"}
```

复制中间的 token 字符串（等号结尾的长串）

### **步骤 3：使用 Token 重置 admin 密码**

执行 `influx user password` 命令，**用刚才提取的管理员 Token 授权**：

重置密码命令：

```
influx user password -n admin -t "你提取的管理员Token"
```

