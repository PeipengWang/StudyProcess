# OpenCloudOS 6.x 安装 MySQL 8.0 完整教程

在 OpenCloudOS 6.x 系统（兼容 CentOS 6/7 生态）上安装 MySQL 8.0（稳定版），以下是可直接执行的完整步骤，包含初始化、配置和启动验证。

## 一、前置准备（清理旧版本 + 配置官方源）

首先清理系统中可能存在的 MariaDB / 旧版 MySQL，避免冲突：

```
# 1. 卸载旧版本（如有）
yum remove -y mariadb-server mariadb mysql-community-server

# 2. 安装依赖
yum install -y wget libaio-devel

# 3. 添加 MySQL 8.0 官方 YUM 源
wget https://dev.mysql.com/get/mysql80-community-release-el8-3.noarch.rpm
rpm -ivh mysql80-community-release-el8-3.noarch.rpm

# 4. 启用 MySQL 8.0 源（OpenCloudOS 6/8 通用）
yum-config-manager --enable mysql80-community
```

## 二、安装 MySQL 8.0

```
# 安装 MySQL 服务器
yum install -y mysql-community-server

# 启动并设置开机自启
systemctl start mysqld
systemctl enable mysqld

# 查看服务状态（确认是否启动成功）
systemctl status mysqld
```

> ✅ 正常启动的标志：输出中包含 `active (running)`。

## 三、初始化配置（核心步骤）

MySQL 8.0 安装后会自动生成临时密码，需先获取并修改：

### 1. 查看临时密码（关键！）

```
grep 'temporary password' /var/log/mysqld.log
```

示例输出：



```
2025-03-11T08:00:00.000000Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: abc123*XYZ
```

复制冒号后的密码（如 `abc123*XYZ`）。

### 2. 登录 MySQL 并修改密码

```
# 登录 MySQL
mysql -uroot -p
# 粘贴上面获取的临时密码，回车登录

# 3. 修改 root 密码（必须包含大小写、数字、特殊字符，否则不符合默认策略）
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourNewPasswd@123';

# 4. 授权 root 远程访问（如需外网连接，否则跳过）
use mysql;
update user set host='%' where user='root';
flush privileges;

# 5. 退出 MySQL
exit;
```

## 四、防火墙 / 端口配置（可选）

如果需要远程连接 MySQL，需开放 3306 端口：



```
# 1. 临时开放 3306 端口
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT

# 2. 永久开放（OpenCloudOS 6 用 iptables 持久化）
service iptables save
service iptables restart

# （如用 firewalld 则执行）
# firewall-cmd --add-port=3306/tcp --permanent
# firewall-cmd --reload
```

## 五、验证安装

```
# 重新登录 MySQL，验证密码是否生效
mysql -uroot -pYourNewPasswd@123

# 查看 MySQL 版本（确认安装成功）
select version();
# 输出：8.0.x 即为成功
```

## 六、常见问题解决

### 1. 安装时报错 "GPG 密钥失败"

```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```

### 2. 临时密码为空 / 找不到

删除初始化文件后重新启动：

```
rm -rf /var/lib/mysql/*
systemctl restart mysqld
grep 'temporary password' /var/log/mysqld.log
```

### 3. 密码设置提示强度不够

临时降低密码策略（测试环境用）：

```
set global validate_password.policy=0;
set global validate_password.length=6;
```

## 总结

1. OpenCloudOS 6.x 安装 MySQL 8.0 需先配置官方 YUM 源，清理旧版 MariaDB 避免冲突；
2. 核心步骤：安装 → 启动服务 → 获取临时密码 → 修改密码 → （可选）开放远程访问；
3. 生产环境需注意：密码设置高强度、限制 root 远程访问、开启防火墙、定期备份数据。