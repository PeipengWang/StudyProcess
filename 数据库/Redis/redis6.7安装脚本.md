# OpenCloudOS 6.x 安装 Redis 完整教程

Redis 推荐通过**源码编译安装**（可自定义版本和配置），以下是适配 OpenCloudOS 6.x 的完整步骤。

## 一、前置准备（安装编译依赖）

bash



运行









```
# 1. 安装编译工具和依赖
yum install -y gcc gcc-c++ make wget jemalloc-devel

# 2. 创建 Redis 安装目录和数据目录
mkdir -p /usr/local/redis /data/redis /var/log/redis
chmod -R 755 /data/redis /var/log/redis
```

## 二、下载并编译 Redis 源码

```
# 1. 下载 Redis 6.2.14（稳定版，适配 OpenCloudOS 6.x）
cd /usr/local/src
wget https://download.redis.io/releases/redis-6.2.14.tar.gz

# 2. 解压源码包
tar -zxvf redis-6.2.14.tar.gz
cd redis-6.2.14

# 3. 编译（指定 jemalloc 内存分配器，性能更优）
make MALLOC=jemalloc

# 4. 安装到指定目录
make PREFIX=/usr/local/redis install
```

## 三、配置 Redis（核心优化）

### 1. 复制配置文件并修改

```
# 复制默认配置文件到安装目录
cp /usr/local/src/redis-6.2.14/redis.conf /usr/local/redis/

# 编辑配置文件（关键优化）
vi /usr/local/redis/redis.conf
```

### 2. 修改以下核心配置（替换默认值）

```
# 1. 守护进程模式（后台运行）
daemonize yes

# 2. 绑定IP（0.0.0.0 允许所有IP访问，生产环境建议指定内网IP）
bind 0.0.0.0

# 3. 端口（默认6379，可自定义）
port 6379

# 4. 数据目录（对应之前创建的目录）
dir /data/redis

# 5. 日志文件
logfile /var/log/redis/redis.log

# 6. 设置密码（必填！避免未授权访问）
requirepass YourRedisPasswd@123

# 7. 最大内存限制（根据服务器配置设置，如 2GB）
maxmemory 2gb
maxmemory-policy allkeys-lru

# 8. 禁用保护模式（允许远程访问）
protected-mode no

# 9. 持久化配置（可选，开启RDB+AOF）
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfilename "appendonly.aof"
```

## 四、配置 Redis 系统服务（开机自启）

```
# 1. 创建 systemd 服务文件
vi /usr/lib/systemd/system/redis.service
```

### 2. 写入以下内容

```
[Unit]
Description=Redis Server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/redis.conf
ExecStop=/usr/local/redis/bin/redis-cli -a YourRedisPasswd@123 shutdown
Restart=on-failure
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

### 3. 重载服务并启动 Redis

```
# 1. 重载 systemd 配置
systemctl daemon-reload

# 2. 启动 Redis
systemctl start redis

# 3. 设置开机自启
systemctl enable redis

# 4. 查看服务状态（确认 active (running)）
systemctl status redis
```

## 五、验证 Redis 安装

```
# 1. 登录 Redis 客户端（输入配置的密码）
/usr/local/redis/bin/redis-cli -a YourRedisPasswd@123

# 2. 测试命令
127.0.0.1:6379> ping
# 输出 PONG 即为成功

127.0.0.1:6379> set test "hello redis"
127.0.0.1:6379> get test
# 输出 "hello redis"

# 3. 退出客户端
127.0.0.1:6379> exit
```

## 六、防火墙配置（可选，允许远程访问）



```
# 1. 临时开放 6379 端口
iptables -A INPUT -p tcp --dport 6379 -j ACCEPT

# 2. 永久开放（OpenCloudOS 6.x 持久化 iptables）
service iptables save
service iptables restart

# （如用 firewalld 则执行）
# firewall-cmd --add-port=6379/tcp --permanent
# firewall-cmd --reload
```

## 七、常见问题解决

### 1. 编译报错 "cc: command not found"

```
# 安装 gcc 编译器
yum install -y gcc gcc-c++
```

### 2. 启动失败 "Address already in use"

```
# 查看 6379 端口占用
netstat -tulpn | grep 6379

# 杀死占用进程（替换 PID 为实际进程号）
kill -9 PID

# 重启 Redis
systemctl restart redis
```

### 3. 远程连接失败

- 确认 `redis.conf` 中 `bind 0.0.0.0` 且 `protected-mode no`；
- 确认防火墙已开放 6379 端口；
- 确认密码输入正确。

## 总结

1. OpenCloudOS 6.x 安装 Redis 优先选择源码编译，兼容性更好；
2. 核心步骤：编译安装 → 配置优化（密码、内存、持久化）→ 配置系统服务 → 启动验证；
3. 生产环境注意：设置强密码、限制绑定 IP、配置最大内存、开启持久化，避免数据丢失和未授权访问。