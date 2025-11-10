------

###  一、修改配置文件允许外部访问

编辑你的配置文件：

```bash
vim /home/redis-7.4.6/redis.conf
```

找到或搜索以下行（大约在前几行）：

```bash
bind 127.0.0.1
```

把它改成：

```bash
bind 0.0.0.0
```

 这表示 Redis 监听所有网卡地址。

然后确认下面这一行：

```bash
protected-mode yes
```

如果你确实需要让外部机器访问（比如远程测试或其他服务器连接），改成：

```bash
protected-mode no
```

>  **强烈建议同时设置密码！**
>  否则任何人都可以访问你的 Redis。

在配置文件中找到：

```bash
# requirepass foobared
```

取消注释并改成你自己的密码，例如：

```bash
requirepass MyStrongPass123!
```

------

###  二、重启 Redis 服务

```bash
sudo systemctl daemon-reload
sudo systemctl restart redis
```

检查状态：

```bash
sudo systemctl status redis
```

此时输出中应该是：

```
/usr/local/bin/redis-server 0.0.0.0:6379
```

------

###  三、检查防火墙

如果仍然无法连接，请检查防火墙或安全组：

```bash
sudo firewall-cmd --add-port=6379/tcp --permanent
sudo firewall-cmd --reload
```

或（使用 `iptables`）：

```bash
sudo iptables -A INPUT -p tcp --dport 6379 -j ACCEPT
```

------

###  四、测试远程连接

在远程机器上用：

```bash
redis-cli -h <你的服务器IP> -p 6379 -a MyStrongPass123!
```

如果能进入 Redis CLI，就说明外部访问配置成功。

------

是否希望我帮你写一个**最安全的最小可行配置示例**（允许外部访问但有密码、限制特定 IP）？