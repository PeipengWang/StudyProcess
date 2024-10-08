

influxdb2版本包含两个包，一个是二进制安装包

## influxdb2安装包

下载二进制安装包

```
wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.7.1-linux-amd64.tar.gz
```

解压

```
tar xvzf influxdb2-2.7.1-linux-amd64.tar.gz
```

直接下载启动即可启动

```
./influxd
```

或者，如果你已经将 `influxd` 放入 `/usr/bin/` 或 `/usr/local/bin/` 目录下，可以直接运行：

```
sudo cp influxdb2-2.7.1-linux-amd64/influxd /usr/local/bin/
influxd
```

你可以通过 `curl` 或浏览器访问 InfluxDB 的 `8086` 端口来验证服务是否成功启动：

```
curl http://localhost:8086/health
```

## 使用 InfluxDB CLI 工具

cli工具下载

```
wget https://dl.influxdata.com/influxdb/releases/influxdb2-client-2.x.x-linux-amd64.tar.gz
```

解压后，将 `influx` CLI 工具移动到系统的可执行路径中：

```
tar xvzf influxdb2-client-2.x.x-linux-amd64.tar.gz
sudo cp influxdb2-client-2.x.x-linux-amd64/influx /usr/local/bin/
```

在 **InfluxDB 2.x** 中，用户管理的命令和选项有所变化，`--password` 这样的选项可能已经被移除或替换。要更新用户密码，正确的命令格式应该是：

```
influx user password --name <username>
```