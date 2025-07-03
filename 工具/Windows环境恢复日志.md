git恢复：





## mysql环境恢复

如果你没安装系统服务，也可在命令行模式定位到mysql下的bin目录里，输入：

```
mysqld --install mysql
net start mysql
```

## Mysql时区错误问题怎么解决

**1 可以通过修改my.cnf
**在 [mysqld] 之下加
default-time-zone=timezone
来修改时区。如：
default-time-zone = '+8:00'
修改完了记得记得重启msyql
注意一定要在 [mysqld] 之下加！！！！ ，否则会出现 unknown variable 'default-time-zone=+8:00'

