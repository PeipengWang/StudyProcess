# 麒麟操作系统linux安装gitlab

## 1. 下载[gitlab](https://so.csdn.net/so/search?q=gitlab&spm=1001.2101.3001.7020)

------

https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/8/gitlab-ce-15.11.0-ce.0.el8.x86_64.rpm

## 2.安装基础依赖

```
yum -y install curl openssh-server openssh-clients postfix cronie policycoreutils-
```

ps:如果报错"错误：没有任何匹配: policycoreutils-python"，执行以下语句

```
dnf -y install postfix
dnf list all | grep policycoreutils
dnf -y install python3-policycoreutils
```

### 3. 安装gitlab

rpm -ivh gitlab-ce-15.11.0-ce.0.el8.x86_64.rpm

出现以下界面说明安装成功：

### 4. 修改端口配置

------

```shell
vim /etc/gitlab/gitlab.rb
#修改external_url字段，改为当前服务器的ip及端口
external_url 'http://192.168.4.39:8888'
```