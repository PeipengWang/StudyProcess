# 神通数据库安装启动流程--linux命令安装

## 1、解压安装

解压压缩包

进入安装目录进行授权

```shell
chmod 755 setup
```

执行安装命令

```shell
./setup -console
```



## 2、选项

0--选择中文

1--继续安装

1--接受许可协议

选择安装路径

1--输入agent密码--默认szoscar55

1--注册HA服务

0--选择jre1.8版本

实例名称--OSRDB

选择编码方式--UTF-8

出现下面表示安装成功

```shell
[ Processing package:  (48/48) ]
[ Unpacking finished ]
Installation finished
Installation was successful
Application installed on /opt/ShenTong
[ Writing the uninstaller data ... ]
[ Console installation done ]

```

## 3、检查启动

检查环境变量是否设置完成

```shell
 cat /etc/profile
```

会发现增加了如下环境变量

```
#####################################
# add environment by shentong db 2024年 06月 13日 星期四 09:54:24 CST
SZ_OSCAR_HOME=/opt/ShenTong
export SZ_OSCAR_HOME

PATH=/opt/ShenTong/bin:/opt/ShenTong/dbstudio:${PATH}
PATH=/opt/ShenTong/bin:/opt/ShenTong/datamigrate:${PATH}
export PATH

LD_LIBRARY_PATH=/opt/ShenTong/bin:/opt/ShenTong/jre/lib/amd64:${LD_LIBRARY_PATH}
export LD_LIBRARY_PATH

#####################################
```

使其生效

```shell
source /etc/profile
```

启动代理服务

```shell
/etc/init.d/oscardb_OSRDBd start
/etc/init.d/oscaragentd start
```

出现下面代表成功

```
# oscar -o install -d OSRDB
Created symlink from /etc/systemd/system/graphical.target.wants/oscardb_OSRDBd.service to /usr/lib/systemd/system/oscardb_OSRDBd.service.
service type : systemctl
Database service was successfully installed!
```

## 4、进入数据库

```shell
isql
```

输入默认默认用户的默认密码

szoscar55

即可进入sql操作界面
