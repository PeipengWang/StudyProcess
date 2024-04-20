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
