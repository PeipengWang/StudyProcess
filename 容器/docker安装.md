# 安装Docker
Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

Docker CE 分为 stable test 和 nightly 三个更新频道。

官方网站上有各种环境下的 安装指南，这里主要介绍 Docker CE 在 CentOS上的安装。

## 1.CentOS安装Docker
Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10， CentOS 7 满足最低内核的要求，所以我们在CentOS 7安装Docker。

### 1.1.卸载（可选）
如果之前安装过旧版本的Docker，可以使用下面命令卸载：

yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce

### 1.2安装docker
首先需要大家虚拟机联网，安装yum工具

yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
然后更新本地镜像源：

### 1.3 设置docker镜像源

yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    

sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
然后输入命令：

yum install -y docker-ce
docker-ce为社区免费版本。稍等片刻，docker即可安装成功。

或者指定版本安装

yum install --setopt=obsoletes=0 docker-ce-19.03.6 docker-ce-cli-19.03.6 containerd.io -y

### 1.4 启动docker
Docker应用需要用到各种端口，逐一去修改防火墙设置。非常麻烦，因此建议大家直接关闭防火墙！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！

#### 1.4.1 关闭防火墙

systemctl stop firewalld

禁止开机启动防火墙

systemctl disable firewalld

#### 1.4.2 通过命令启动docker：

systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

systemctl restart docker  # 重启docker服务

### 1.3 查看docker版本：

docker -v

### 1.4 配置镜像加速

方式一：cmd命令方式方式

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://v6s6gzx9.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

方式二：修改配置

```shell
sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
vim /etc/systemd/system/docker.service
```

修改：

```vim
ExecStart=/usr/bin/dockerd --registry-mirror=你的加速地址
```

重启生效

```shell
 systemctl daemon-reload
service docker restart
```





```
sudo mkdir -p /etc/systemd/system/docker.service.d


sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://127.0.0.1:8123"
Environment="HTTPS_PROXY=http://127.0.0.1:8123"




```

```
vim ~/.docker/config.json
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://172.17.0.1:8123",
     "httpsProxy": "http://172.17.0.1:8123",
     "noProxy": "localhost,127.0.0.1,.daocloud.io"
   }
 }
}





```

